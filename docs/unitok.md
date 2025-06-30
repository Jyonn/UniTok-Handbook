# UniTok

!!! note "Download the required tiny dataset for demonstration!"

    Please download UniTok-tokenized tiny dataset and unzip the file before running the code snippets. [:material-download:](https://s.6-79.cn/yYU10e/sample-ut.zip)  


The `UniTok` class is the main class of the UniTok library. 
It is used to manage the tokenization process for a specific dataframe.

`UniTok` has three states: **Initialized**, **Tokenized**, and **Organized**.

- **Initialized**: The initial state after creating a UniTok instance.
- **Tokenized**: Achieved after applying tokenization to the dataframe or loading a tokenized dataframe.
- **Organized**: Reached after combining multiple dataframes via operations like union and filter.
- The **disk** state is not a real state but a representation of the saved tokenized dataframe.
- An **organized** unitok can move back to the **tokenized** state only by loading the saved dataframe from **disk**.

The following graph illustrates the transitions between these states:

``` mermaid
graph LR
  A[Initialized] --> |tokenize| B[Tokenized];
  B --> |union/filter| C[Organized];
  C --> |save| D[Disk];
  B --> |save| D;
  D --> |UniTok.load| B;
```

## Constructor

### `CLASS` unitok.UniTok()

Initialize an empty UniTok object.

```python
from unitok import UniTok

with UniTok() as ut:
    print(ut.status)  # <initialized>
```

## Attributes

| Attribute          | Type                              | Description                                                                          |
|--------------------|-----------------------------------|--------------------------------------------------------------------------------------|
| `data`             | Dict\[str, list\]                 | The tokenized dataframe.                                                             |
| `meta`             | unitok.Meta                       | The metadata of the UniTok object, including the vocabularies, tokenizers, and jobs. |
| `key_job`          | unitok.Job                        | The key job of the UniTok object.                                                    |
| `save_dir`         | str                               | The directory to save the tokenized dataframe.                                       |
| `_legal_indices`   | List\[int\]                       | The legal indices of the tokenized dataframe.                                        |
| `_legal_flags`     | List\[bool\]                      | The legal flags of the tokenized dataframe.                                          |
| `_indices_is_init` | bool                              | Whether the indices are initialized.                                                 |
| `_sample_size`     | int                               | The real sample size (including legal and illegal) of the tokenized dataframe.       |
| `_union_type`      | unitok.Symbol                     | The type of the union operation, can be either `Symbols.soft` or `Symbols.hard`.     |
| `_soft_unions`     | Dict\[str, Set\[unitok.UniTok\]\] | The soft unions of the UniTok object.                                                |

## Properties

### `PROPERTY` is_soft_union -> bool

Whether the UniTok object is a soft union.

```python
from unitok import UniTok

with UniTok() as ut:
    print(ut.is_soft_union)  # False, the initial _union_type is None
    
    ut.set_union_type(soft_union=True)
    print(ut.is_soft_union)  # True
```

### `PROPERTY` is_hard_union -> bool

Whether the UniTok object is a hard union.

```python
from unitok import UniTok

with UniTok() as ut:
    print(ut.is_hard_union)  # False, the initial _union_type is None
    
    ut.set_union_type(soft_union=False)
    print(ut.is_hard_union)  # True
```

### `PROPERTY` filepath -> str

The filepath of the tokenized dataframe.

```python
from unitok import UniTok

with UniTok.load('sample-ut/item') as ut:
    print(ut.filepath)  # 'sample-ut/item/data.pkl'

# an initialized unitok has no filepath until it is saved
```

## Magic Methods

### `METHOD` \_\_enter\_\_() -> unitok.UniTok
### `METHOD` \_\_exit\_\_() -> None

Enter or exit the context manager or UniTok Space.
The definition of the vocabularies, tokenizers, and jobs in `with UniTok() as ut:` will be stored in the `ut` space.
The next time when you want to pass the vocabularies, tokenizers, and jobs to the UniTok methods, you can pass vocabulary names, tokenizer IDs, and job names instead.

The UniTok context manager can not be nested. 

```python
from unitok import UniTok, Vocab, EntityTokenizer

with UniTok() as ut:
    vocab = Vocab(name='fruits')  # 'fruits' vocab has been registered in the ut space
    vocab.extend(['apple', 'banana', 'cherry'])

    fruit_tokenizer = EntityTokenizer(vocab='fruits')  # same as EntityTokenizer(vocab=vocab)
    print(list(fruit_tokenizer.vocab))  # ['apple', 'banana', 'cherry']

with UniTok() as another_ut:
    another_fruit_tokenizer = EntityTokenizer(vocab='fruits')  # will create a new vocabulary named 'fruits' in the another_ut space
    print(list(another_fruit_tokenizer.vocab))  # []

    with UniTok() as nested_ut:  # ValueError: Space is already locked to another_ut
        ...
```

### `METHOD` \_\_getitem\_\_() -> dict

!!! warning "State Required"

    :material-check: **Tokenized**
    :material-check: **Organized**
    :material-close: **Initialized**

Index the item, and select the attributes (or jobs), and return the data in a dict format.

| Parameter | Default | Type                        | Example | Description                                    |
|-----------|---------|-----------------------------|---------|------------------------------------------------|
| `index`   | `N/A`   | Please refer to the example | `0`     | The index or slice of the tokenized dataframe. |

```python
from unitok import UniTok

with UniTok.load('sample-ut/item') as ut:
    # 1. Select all the attributes from the first item using item index 
    print(ut[0])  
    # {
    #   'category': 0, 
    #   'abstract@bert': [4497, 1996, 14960, ...], 
    #   'title@llama': [450, 1771, 4167, ...], 
    #   'title@bert': [1996, 9639, 3035, ...], 
    #   'abstract@llama': [1383, 459, 278, ...], 
    #   'subcategory': 0, 
    #   'nid': 0
    # }

    # 2. Select all the attributes from the first item using item key value
    print(ut.key_job.tokenizer.vocab[0])  # 'N88753', nid of the first item
    print(ut['N88753'])  # same as ut[0]

    # 3. Select the 'title@bert' attribute from the first item
    print(ut[0, 'title@bert'])  # {'title@bert': [1996, 9639, 3035, ...]}
    print(ut['N88753', 'title@bert'])  # same as ut[0, 'title@bert']
    
    # 4. Select the attribute tokenized by the 'title@bert' job from the first item
    job = ut.meta.jobs['title@bert']
    print(ut[0, job])  # same as ut[0, 'title@bert']
    
    # 5. Select the 'title@bert' and 'abstract@bert' attributes from the first item
    print(ut[0, ('title@bert', 'abstract@bert')])  # {'title@bert': [1996, 9639, 3035, ...], 'abstract@bert': [4497, 1996, 14960, ...]}

    # 6. Select the attribute tokenized by the specific tokenizer from the first item
    tokenizer = job.tokenizer
    print(ut[0, tokenizer])  # same as ut[0, ('title@bert', 'abstract@bert')]
```

### `METHOD` \_\_len\_\_() -> int

Return the legal sample size of the tokenized dataframe.

```python
from unitok import UniTok

with UniTok.load('sample-ut/item') as ut:
    print(len(ut))  # 10
    print(ut._sample_size)  # 10
    
    ut.filter(lambda x: x % 2 == 0, job='nid')  # filter the even indices
    print(len(ut))  # 5
    print(ut._sample_size)  # 10
```

### `METHOD` \_\_iter\_\_() -> Iterator

Iterate over the samples at the legal indices of the tokenized dataframe.

```python
from unitok import UniTok

with UniTok.load('sample-ut/item') as ut:
    for sample in ut:
        print(sample)  # {'nid': ..., 'category': ..., 'title@bert': ..., ...} 
```

## Methods

### `METHOD` set_union_type() -> None

Set the union type of the UniTok object. Each UniTok can only set once.

| Parameter    | Default | Type | Example | Description                             |
|--------------|---------|------|---------|-----------------------------------------|
| `soft_union` | `N/A`   | bool | `True`  | Whether the union type is soft or hard. |

```python
from unitok import UniTok

with UniTok() as ut:
    print(ut.is_soft_union)  # False, the initial _union_type is None
    
    ut.set_union_type(soft_union=True)
    print(ut.is_soft_union)  # True

    ut.set_union_type(soft_union=False)  # ValueError: Union type is already set
```

### `METHOD` init_indices() -> None

!!! warning "State Required"

    :material-check: **Tokenized**
    :material-check: **Organized**
    :material-close: **Initialized**

Initialize the indices of the tokenized dataframe. Legal indices will be reset.

```python
from unitok import UniTok

with UniTok.load('sample-ut/item') as ut:  # init_indices will be automatically called
    print(len(ut))  # 10
    
    ut.filter(lambda x: x % 2 == 0, job='nid')  # filter the even indices
    print(len(ut))  # 5
    
    ut.init_indices()
    print(len(ut))  # 10
```

### `METHOD` load() -> None

!!! danger "State Transfer"

    **Disk** -> **Tokenized**

Load the tokenized dataframe from disk.

| Parameter       | Default | Type | Example            | Description                                                                                                                                                                                |
|-----------------|---------|------|--------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `save_dir`      | `N/A`   | str  | `'sample-ut/item'` | The directory to save the tokenized dataframe.                                                                                                                                             |
| `tokenizer_lib` | `None`  | str  | `None`             | When using custom tokenizers, you can set the directory of the tokenizer files to reconstruct these tokenizers. If not set, the unrecognized tokenizers will be set to `UnknownTokenizer`. |

```python
from unitok import UniTok

with UniTok.load('sample-ut/item') as ut:
    ...
```

### `METHOD` save() -> None

!!! warning "State Required"

    :material-check: **Tokenized**
    :material-check: **Organized**
    :material-close: **Initialized**

Save the tokenized dataframe to disk.

```python
from unitok import UniTok

with UniTok.load('sample-ut/item') as ut:
    ut.save('sample-ut/another-item')

with UniTok.load('sample-ut/another-item') as another_ut:
    ...
```

### `METHOD` add_job() -> None

!!! warning "State Required"

    :material-check: **Initialized**
    :material-check: **Tokenized**
    :material-close: **Organized**

Add a tokenization job to the UniTok object.

| Parameter   | Default | Type                         | Example          | Description                                                                                                                                                                                                        |
|-------------|---------|------------------------------|------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `tokenizer` | `N/A`   | Union[unitok.Tokenizer, str] | `bert_tokenizer` | The tokenizer to tokenize the current column. If a string (`tokenizer_id`) is provided, the current space will be searched for the tokenizer.                                                                      |
| `column`    | `N/A`   | str                          | `'title'`        | The column name to tokenize in the dataframe.                                                                                                                                                                      |
| `name`      | `None`  | Optional[str]                | `'title@bert'`   | The tokenized column name. If not provided, the column name will be used. It should be unique in the current unitok space.                                                                                         |
| `truncate`  | `None`  | Optional[int]                | `50`             | The maximum length of the tokenized sequence. When the current tokenizer does not return a list, set it to `None`. To keep the full sequence, set it to `0`. To truncate from the end, set it to a negative value. |
| `key`       | `False` | bool                         | `False`          | Whether to use the tokenized column as the key/primary column. The key column will be used as the index of the data.                                                                                               |

### `METHOD` tokenize() -> self

!!! warning "State Required"

    :material-check: **Initialized**
    :material-check: **Tokenized**
    :material-close: **Organized**

!!! danger "State Transfer"

    **Initialized** -> **Tokenized**

Tokenize the dataframe based on the added jobs. The key job should be set before tokenization.

| Parameter | Default | Type         | Example | Description                |
|-----------|---------|--------------|---------|----------------------------|
| `df`      | `N/A`   | pd.DataFrame | `data`  | The dataframe to tokenize. |

```python
import pandas as pd

from unitok import UniTok, BertTokenizer, TransformersTokenizer, EntityTokenizer

item = pd.read_csv(
    filepath_or_buffer='news-sample.tsv',
    sep='\t',
    names=['nid', 'category', 'subcategory', 'title', 'abstract', 'url', 't_ent', 'a_ent'],
    usecols=['nid', 'category', 'subcategory', 'title', 'abstract'],
)
item['abstract'] = item['abstract'].fillna('')  # Handle missing values

with UniTok() as item_ut:
    bert_tokenizer = BertTokenizer(vocab='bert')
    llama_tokenizer = TransformersTokenizer(vocab='llama', key='huggyllama/llama-7b')

    item_ut.add_job(tokenizer=EntityTokenizer(vocab='item_id'), column='nid', key=True)
    item_ut.add_job(tokenizer=bert_tokenizer, column='title', name='title@bert', truncate=20)
    item_ut.add_job(tokenizer=llama_tokenizer, column='title', name='title@llama', truncate=20)
    item_ut.add_job(tokenizer=bert_tokenizer, column='abstract', name='abstract@bert', truncate=50)
    item_ut.add_job(tokenizer=llama_tokenizer, column='abstract', name='abstract@llama', truncate=50)
    item_ut.add_job(tokenizer=EntityTokenizer(vocab='category'), column='category')
    item_ut.add_job(tokenizer=EntityTokenizer(vocab='subcategory'), column='subcategory')

    item_ut.tokenize(item)
    print(item_ut.status)  # <tokenized>
```

### `METHOD` union() -> None

!!! warning "State Required"

    :material-check: **Tokenized**
    :material-check: **Organized**
    :material-close: **Initialized**

!!! danger "State Transfer"

    **Tokenized** -> **Organized**

!!! note "Soft Union and Hard Union"

    - **Soft Union**: Fast. The union UniToks will be saved in the `_soft_unions` attribute. When retrieving the data by `ut[index]`, the data will be combined on the fly. `ut.data` will not be changed.
    
    - **Hard Union**: Slow. `ut.data` will be expanded to include the union data.

    Both soft and hard unions will update the `ut.meta.vocabularis`. All the tokenizers from the other UniTok will be switched into the `UnionTokenizer` class. All the union jobs will be tagged with `job.from_union = True`.

    However, there would be no differences between these two union types after saving the UniTok and loading it again. 

Union the current UniTok object with another UniTok object.

| Parameter    | Default | Type          | Example    | Description                                                                                                                          |
|--------------|---------|---------------|------------|--------------------------------------------------------------------------------------------------------------------------------------|
| `other`      | `N/A`   | unitok.UniTok | `other_ut` | The other UniTok object to union with the current UniTok object.                                                                     |
| `soft_union` | `True`  | bool          | `True`     | Whether to perform a soft union.                                                                                                     |
| `union_key`  | `None`  | str           | `None`     | The column in current UniTok that used as index (key_job) in other UniTok. If not set, the key job of the other UniTok will be used. |

```python
from unitok import UniTok

user_ut = UniTok.load('sample-ut/user')
inter_ut = UniTok.load('sample-ut/interaction')

# => {'uid': 0, 'history': [0, 1, 2]}
print(user_ut[0])

# => {'uid': 0, 'nid': 7, 'index': 0, 'click': 1}
print(inter_ut[0])

with inter_ut:
    inter_ut.union(user_ut)

    # => {'index': 0, 'click': 1, 'uid': 0, 'nid': 7, 'history': [0, 1, 2]}
    print(inter_ut[0])
```

### `METHOD` summarize() -> None

!!! warning "State Required"

    :material-check: **Tokenized**
    :material-check: **Organized**
    :material-close: **Initialized**

Summarize the tokenized dataframe.

```python
from unitok import UniTok

with UniTok.load('sample-ut/item') as ut:
    ut.summarize()
```

```text
UniTok (v4), Data (v4)
Sample Size: 10
ID Column: nid
```

| Tokenizer             | Tokenizer ID | Column Mapping             | Vocab                | Max Length |
|-----------------------|--------------|----------------------------|----------------------|------------|
| TransformersTokenizer | auto_2VN5Ko  | abstract -> abstract@llama | llama (size=32024)   | 50         |
| EntityTokenizer       | auto_C0b9Du  | subcategory -> subcategory | subcategory (size=8) | N/A        |
| TransformersTokenizer | auto_2VN5Ko  | title -> title@llama       | llama (size=32024)   | 20         |
| EntityTokenizer       | auto_4WQYxo  | category -> category       | category (size=4)    | N/A        |
| BertTokenizer         | auto_Y9tADT  | abstract -> abstract@bert  | bert (size=30522)    | 46         |
| BertTokenizer         | auto_Y9tADT  | title -> title@bert        | bert (size=30522)    | 16         |
| EntityTokenizer       | auto_qwQALc  | nid -> nid                 | nid (size=10)        | N/A        |

### `METHOD` pack() -> dict

!!! warning "State Required"

    :material-check: **Tokenized**
    :material-check: **Organized**
    :material-close: **Initialized**

Pack the tokenized dataframe into a dictionary.

!!! note "Comparison with `ut[index]`"

    The index of the `pack` method is the absolute index: `ut._sample_size > index >= 0`.
    The index of the `ut[index]` method is the legal index: `len(ut) > index >= 0`.

| Parameter | Default | Type | Example | Description                                    |
|-----------|---------|------|---------|------------------------------------------------|
| `index`   | `N/A`   | int  | `0`     | The absolute index of the tokenized dataframe. |

```python
from unitok import UniTok

with UniTok.load('sample-ut/item') as ut:
    ut.filter(lambda x: x % 2 == 0, job='nid')  # filter the even indices
    print(ut[1])  
    # {
    #   'abstract@bert': [2122, 9428, 19741, ...], 
    #   'subcategory': 2, 
    #   'title@llama': [29871, 29945, 29900, ...], 
    #   'nid': 2,
    #   'abstract@llama': [4525, 2833, 11687, ...], 
    #   'category': 2, 
    #   'title@bert': [2753, 5409, 14243, ...]
    # }
    print(ut.pack(1))  
    # {
    #   'abstract@bert': [6207, 1005, 1055, ...], 
    #   'subcategory': 1, 
    #   'title@llama': [5260, 28402, 14866, ...], 
    #   'nid': 1, 
    #   'abstract@llama': [12113, 29915, 29879, ...], 
    #   'category': 1, 
    #   'title@bert': [24547, 22345, 18296, ...]
    # }
```

### `METHOD` select() -> dict

!!! warning "State Required"

    :material-check: **Tokenized**
    :material-check: **Organized**
    :material-close: **Initialized**

Select the attributes (or jobs) of current sample, and return the data in a dict format.

| Parameter  | Default | Type                        | Example                 | Description                          |
|------------|---------|-----------------------------|-------------------------|--------------------------------------|
| `sample`   | `N/A`   | dict                        | `sample`                | The sample to select the attributes. |
| `selector` | `N/A`   | Please refer to the example | `('nid', 'title@bert')` | The attributes to select.            |

\* More usage examples can be found in the `__getitem__` method.

```python
from unitok import UniTok

with UniTok.load('sample-ut/item') as ut:
    sample = ut[0]
    print(ut.select(sample, ('nid', 'title@bert')))  
    # {'nid': 0, 'title@bert': [1996, 9639, 3035, ...]}
```

### `METHOD` filter() -> None

!!! warning "State Required"

    :material-check: **Tokenized**
    :material-check: **Organized**
    :material-close: **Initialized**

!!! danger "State Transfer"

    **Tokenized** -> **Organized**

Filter the tokenized dataframe based on the condition.

| Parameter | Default | Type | Example                | Description                                                                                                      |
|-----------|---------|------|------------------------|------------------------------------------------------------------------------------------------------------------|
| `func`    | `N/A`   | Any  | `lambda x: x % 2 == 0` | The filter function. The function should return `True` to keep the sample, and `False` to remove it.             |
| `job`     | `None`  | str  | `'nid'`                | The column name to apply the filter. If not set, the input of the filter function will be the sample dictionary. |

```python
from unitok import UniTok

with UniTok.load('sample-ut/item') as ut:
    print(len(ut))  # 10
    
    ut.filter(lambda x: x % 2 == 0, job='nid')  # filter the even indices
    print(len(ut))  # 5
    
    ut.init_indices()
    print(len(ut))  # 10

    ut.filter(lambda x: x['nid'] <= 2)  # filter the nids less than or equal to 2
    print(len(ut))  # 3
```

### `METHOD` retruncate() -> None

!!! warning "State Required"

    :material-check: **Tokenized**
    :material-check: **Organized**
    :material-close: **Initialized**

Re-truncate the tokenized sequences based on the new truncate value.

| Parameter  | Default | Type                   | Example        | Description                                |
|------------|---------|------------------------|----------------|--------------------------------------------|
| `job`      | `N/A`   | Union[str, unitok.Job] | `'title@bert'` | The job name or job object to re-truncate. |
| `truncate` | `N/A`   | int                    | `10`           | The new truncate value.                    |

```python
from unitok import UniTok

with UniTok.load('sample-ut/item') as ut:
    print(ut[0, 'title@bert'])  # {'title@bert': [1996, 9639, 3035, ...]}
    ut.retruncate('title@bert', 2)
    print(ut[0, 'title@bert'])  # {'title@bert': [1996, 9639]}
```

### `METHOD` remove_job() -> None

Delete the job from the UniTok object.

| Parameter | Default | Type            | Example        | Description                           |
|-----------|---------|-----------------|----------------|---------------------------------------|
| `job`     | `N/A`   | Union[str, Job] | `'title@bert'` | The job name or job object to remove. |

```python
from unitok import UniTok

with UniTok.load('sample-ut/item') as ut:
    print(ut.data.keys())  # dict_keys(['nid', 'category', 'subcategory', 'title@bert', 'abstract@llama', 'title@llama', 'abstract@bert'])
    ut.remove_job('title@bert')
    print(ut.data.keys())  # dict_keys(['nid', 'category', 'subcategory', 'abstract@llama', 'title@llama', 'abstract@bert'])
```