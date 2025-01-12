# Quick Start

## Data Preparation

Base on the [MIND dataset](https://msnews.github.io/), we construct a tiny dataset for demonstration purposes, including

- **news-sample.tsv**, a sample of news articles. [:material-download:](https://s.6-79.cn/QpC2ss/news-sample.tsv)
- **user-sample.tsv**, a sample of user profiles. [:material-download:](https://s.6-79.cn/HXGMNO/user-sample.tsv)
- **interaction-sample.tsv**, a sample of user-item interactions. [:material-download:](https://s.6-79.cn/uZlLdl/interaction-sample.tsv)
- **sample-ut.zip**, the tokenized data using UniTok. [:material-download:](https://s.6-79.cn/yYU10e/sample-ut.zip) (Please unzip it before using)

## Loading Data

```python
import pandas as pd

item = pd.read_csv(
    filepath_or_buffer='news-sample.tsv',
    sep='\t',
    names=['nid', 'category', 'subcategory', 'title', 'abstract', 'url', 't_ent', 'a_ent'],
    usecols=['nid', 'category', 'subcategory', 'title', 'abstract'],
)
item['abstract'] = item['abstract'].fillna('')  # Handle missing values

user = pd.read_csv(
    filepath_or_buffer='user-sample.tsv',
    sep='\t',
    names=['uid', 'history'],
)

interaction = pd.read_csv(
    filepath_or_buffer='interaction-sample.tsv',
    sep='\t',
    names=['uid', 'nid', 'click'],
)
```

## Defining Tokenization Jobs

```python
from unitok import UniTok, Vocab
from unitok.tokenizer import BertTokenizer, TransformersTokenizer, EntityTokenizer, SplitTokenizer, DigitTokenizer

item_vocab = Vocab(name='nid')  # will be used across datasets
user_vocab = Vocab(name='uid')  # will be used across datasets

with UniTok() as item_ut:
    bert_tokenizer = BertTokenizer(vocab='bert')
    llama_tokenizer = TransformersTokenizer(vocab='llama', key='huggyllama/llama-7b')

    item_ut.add_job(tokenizer=EntityTokenizer(vocab=item_vocab), column='nid', key=True)
    item_ut.add_job(tokenizer=bert_tokenizer, column='title', name='title@bert', truncate=20)
    item_ut.add_job(tokenizer=llama_tokenizer, column='title', name='title@llama', truncate=20)
    item_ut.add_job(tokenizer=bert_tokenizer, column='abstract', name='abstract@bert', truncate=50)
    item_ut.add_job(tokenizer=llama_tokenizer, column='abstract', name='abstract@llama', truncate=50)
    item_ut.add_job(tokenizer=EntityTokenizer(vocab='category'), column='category')
    item_ut.add_job(tokenizer=EntityTokenizer(vocab='subcategory'), column='subcategory')

with UniTok() as user_ut:
    user_ut.add_job(tokenizer=EntityTokenizer(vocab=user_vocab), column='uid', key=True)
    user_ut.add_job(tokenizer=SplitTokenizer(vocab=item_vocab, sep=','), column='history', truncate=30)

with UniTok() as inter_ut:
    inter_ut.add_index_job(name='index')
    inter_ut.add_job(tokenizer=EntityTokenizer(vocab=user_vocab), column='uid')
    inter_ut.add_job(tokenizer=EntityTokenizer(vocab=item_vocab), column='nid')
    inter_ut.add_job(tokenizer=DigitTokenizer(vocab='click', vocab_size=2), column='click')
```

## Tokenizing and Saving Data

```python
item_ut.tokenize(item).save('sample-ut/item')
item_vocab.deny_edit()  # will raise an error if new items are detected in the user or interaction datasets
user_ut.tokenize(user).save('sample-ut/user')
inter_ut.tokenize(interaction).save('sample-ut/interaction')
```

## Combining UniToks

```python
# => {'category': 0, 'nid': 0, 'title@bert': [1996, 9639, 3035, 3870, ...], 'title@llama': [450, 1771, 4167, 10470, ...], 'abstract@bert': [4497, 1996, 14960, 2015, ...], 'abstract@llama': [1383, 459, 278, 451, ...], 'subcategory': 0}
print(item_ut[0])

# => {'uid': 0, 'history': [0, 1, 2]}
print(user_ut[0])

# => {'uid': 0, 'nid': 7, 'index': 0, 'click': 1}
print(inter_ut[0])

with inter_ut:
    inter_ut.union(user_ut)

    # => {'index': 0, 'click': 1, 'uid': 0, 'nid': 7, 'history': [0, 1, 2]}
    print(inter_ut[0])
```

## Glance at the Terminal

```bash
unitok sample-ut/item
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
