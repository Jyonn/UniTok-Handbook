# Job

The `Job` class links one column of the data with one tokenizer. 
Each column of the data can be linked with multiple tokenizers, and each tokenizer can be linked with multiple columns of the data.
Unlike the `Tokenizer` class and the `Vocab` class that can be declared in the default global space, the `Job` class must be declared in a UniTok space.

## Constructor

### `CLASS` unitok.Job()

| Parameter   | Default | Type                           | Example          | Description                                                                                                                                                                                                  |
|-------------|---------|--------------------------------|------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `tokenizer` | `N/A`   | Union\[unitok.Tokenizer, str\] | `bert_tokenizer` | The tokenizer to tokenize the current column. If a string (`tokenizer_id`) is provided, the current space will be searched for the tokenizer.                                                                |
| `column`    | `N/A`   | Union\[str, int\]              | `'title'`        | The column name to tokenize.                                                                                                                                                                                 |
| `name`      | `None`  | Optional\[str\]                | `'title@bert'`   | The tokenized column name. If not provided, the column name will be used. It should be unique in the current space.                                                                                          |
| `truncate`  | `None`  | Optional\[int\]                | `50`             | The maximum length of the tokenized sequence. When current tokenizer does not return list, set it to `None`. To keep the full sequence, set it to `0`. To truncate from the end, set it to a negative value. | 
| `order`     | `-1`    | int                            | `-1`             | The processing order of the job among all jobs. `-1` represents the not processed job. **Please do not set it manually.**                                                                                    |
| `key`       | `False` | bool                           | `False`          | Whether to use the tokenized column as the key/primary column. The key column will be used as the index of the data.                                                                                         |
| `max_len`   | `0`     | Optional\[int\]                | `0`              | The real maximum length of the tokenized sequence. It will be not larger than `abs(truncate)`. **Please do not set it manually.**                                                                            |

```python
import pandas as pd

from unitok import Job, UniTok, BertTokenizer, EntityTokenizer, TransformersTokenizer

data = pd.DataFrame({
    'title': ['The Brands Queen Elizabeth, Prince Charles, and Prince Philip Swear By', 
              'Walmart Slashes Prices on Last-Generation iPads', 
              '50 Worst Habits For Belly Fat'],
    'abstract': ["Shop the notebooks, jackets, and more that the royals can't live without.",
                 "Apple's new iPad releases bring big deals on last year's models.",
                 "These seemingly harmless habits are holding you back and keeping you from shedding that unwanted belly fat for good."],
    'item_id': ["ID001", "ID002", "ID003"]
})

with UniTok() as ut:
    bert_tokenizer = BertTokenizer(vocab='bert')
    llama1_tokenizer = TransformersTokenizer(vocab='llama', key='huggyllama/llama-7B')
    entity_tokenizer = EntityTokenizer(vocab='item_id')
        
    title_bert_job = Job(tokenizer=bert_tokenizer, column='title', name='title@bert', truncate=20)
    abstract_bert_job = Job(tokenizer=bert_tokenizer, column='abstract', name='abstract@bert', truncate=50)
    title_llama1_job = Job(tokenizer=llama1_tokenizer, column='title', name='title@llama1', truncate=20)
    abstract_llama1_job = Job(tokenizer=llama1_tokenizer, column='abstract', name='abstract@llama1', truncate=50)
    item_job = Job(tokenizer=entity_tokenizer, column='item_id', key=True)

    title_job = Job(tokenizer=bert_tokenizer, column='title', name='title@bert', truncate=20)  # ValueError: Conflict object declaration
    another_item_job = Job(tokenizer=entity_tokenizer, column='item_id', name='item_id@entity', key=True)  # ValueError: Key column already exists

outer_title_bert_job = Job(tokenizer=bert_tokenizer, column='title', name='title@bert', truncate=20)  # ValueError: Required UniTok context, please use `with UniTok() as ut:`
```

\* We usually use `ut.add_job()` method to declare a job in the UniTok space. Here we use the constructor for demonstration.

## Attributes

| Attribute    | Type             | Description                                                         |
|--------------|------------------|---------------------------------------------------------------------|
| `tokenizer`  | unitok.Tokenizer | The tokenizer used by the job.                                      |
| `column`     | str              | The column name to tokenize.                                        |
| `name`       | str              | The tokenized column name.                                          |
| `truncate`   | int              | The maximum length of the tokenized sequence.                       |
| `order`      | int              | The processing order of the job among all jobs.                     |
| `slice`      | slice            | The slice of the tokenized sequence, based on the `truncate` value. |
| `key`        | bool             | Whether to use the tokenized column as the key/primary column.      |
| `max_len`    | int              | The real maximum length of the tokenized sequence.                  |
| `from_union` | bool             | Whether the job is from the union of two UniToks.                   |

## Properties

### `PROPERTY` return_list -> bool

Whether the tokenizer should return a list of tokens. 
It is determined by the `truncate` value.
When `truncate` is not `None`, `return_list` is `True`.

```python
import pandas as pd

from unitok import Job, UniTok, BertTokenizer, EntityTokenizer

data = pd.DataFrame({
    'title': ['The Brands Queen Elizabeth, Prince Charles, and Prince Philip Swear By', 
              'Walmart Slashes Prices on Last-Generation iPads', 
              '50 Worst Habits For Belly Fat'],
    'item_id': ["ID001", "ID002", "ID003"]
})

with UniTok() as ut:
    bert_tokenizer = BertTokenizer(vocab='bert')
    entity_tokenizer = EntityTokenizer(vocab='item_id')
        
    title_bert_job = Job(tokenizer=bert_tokenizer, column='title', name='title@bert', truncate=20)
    item_job = Job(tokenizer=entity_tokenizer, column='item_id', key=True)

    print(title_bert_job.return_list)  # True
    print(item_job.return_list)  # False
```

### `PROPERTY` is_processed -> bool

Whether the job has been processed, or the column has been tokenized.
It is determined by the `order` value.
When `order` is `-1`, `is_processed` is `False`.

```python
import pandas as pd

from unitok import UniTok, BertTokenizer, EntityTokenizer

data = pd.DataFrame({
    'title': ['The Brands Queen Elizabeth, Prince Charles, and Prince Philip Swear By', 
              'Walmart Slashes Prices on Last-Generation iPads', 
              '50 Worst Habits For Belly Fat'],
    'item_id': ["ID001", "ID002", "ID003"]
})

with UniTok() as ut:
    bert_tokenizer = BertTokenizer(vocab='bert')
    entity_tokenizer = EntityTokenizer(vocab='item_id')
        
    ut.add_job(tokenizer=bert_tokenizer, column='title', name='title@bert', truncate=20)
    ut.add_job(tokenizer=entity_tokenizer, column='item_id', key=True)
    
print(ut.meta.jobs['title@bert'].is_processed)  # False

ut.tokenize(data)
print(ut.meta.jobs['title@bert'].is_processed)  # True
```

## Methods

### `METHOD` clone() -> unitok.Job

Clone the job. Only used during the `union` operation of two UniToks.

### `METHOD` json() -> Dict[str, Any]

Return the metadata of the job.

```python
from unitok import UniTok, BertTokenizer

with UniTok() as ut:
    bert_tokenizer = BertTokenizer(vocab='bert')
    ut.add_job(tokenizer=bert_tokenizer, column='title', name='title@bert', truncate=20)

print(ut.meta.jobs['title@bert'].json())  
# {
#   'name': 'title@bert', 
#   'column': 'title', 
#   'tokenizer': 'auto_LoGCuf', 
#   'truncate': 20, 
#   'order': -1,
#   'key': False, 
#   'max_len': 0
# }
```