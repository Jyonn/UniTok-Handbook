# Quick Start

## Data Preparation

Base on the [MIND dataset](https://msnews.github.io/), we construct a tiny dataset for demonstration purposes, including

- **news-sample.tsv**, a sample of news articles. [:material-download:](https://s.6-79.cn/QpC2ss/news-sample.tsv)
- **user-sample.tsv**, a sample of user profiles. [:material-download:](https://s.6-79.cn/HXGMNO/user-sample.tsv)
- **interaction-sample.tsv**, a sample of user-item interactions. [:material-download:](https://s.6-79.cn/uZlLdl/interaction-sample.tsv)
- **sample-ut.zip**, the tokenized data using UniTok. [:material-download:](https://s.6-79.cn/yYU10e/sample-ut.zip) (Please unzip it before using)

??? note "news-sample.tsv"

    | nid     | category  | subcategory              | title                                                                  | abstract                                                                                                                                                                                             | url                                           | t_ent                                                                                                                                                                 | a_ent                                                                                                                                                       |
    |---------|-----------|--------------------------|------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
    | N88753  | lifestyle | lifestyleroyals          | The Brands Queen Elizabeth, Prince Charles, and Prince Philip Swear By | Shop the notebooks, jackets, and more that the royals can't live without.                                                                                                                            | https://assets.msn.com/labs/mind/AAGH0ET.html | [{"Label": "Prince Philip, Duke of Edinburgh", "Type": "P", "WikidataId": "Q80976", "Confidence": 1.0, "OccurrenceOffsets": [48], "SurfaceForms": ["Prince Philip"]}] | []                                                                                                                                                          |
    | N45436  | news      | newsscienceandtechnology | Walmart Slashes Prices on Last-Generation iPads                        | Apple's new iPad releases bring big deals on last year's models.                                                                                                                                     | https://assets.msn.com/labs/mind/AABmf2I.html | [{"Label": "IPad", "Type": "J", "WikidataId": "Q2796", "Confidence": 0.999, "OccurrenceOffsets": [42], "SurfaceForms": ["iPads"]}]                                    | [{"Label": "IPad", "Type": "J", "WikidataId": "Q2796", "Confidence": 0.999, "OccurrenceOffsets": [12], "SurfaceForms": ["iPad"]}]                           |
    | N23144  | health    | weightloss               | 50 Worst Habits For Belly Fat                                          | These seemingly harmless habits are holding you back and keeping you from shedding that unwanted belly fat for good.                                                                                 | https://assets.msn.com/labs/mind/AAB19MK.html | [{"Label": "Adipose tissue", "Type": "C", "WikidataId": "Q193583", "Confidence": 1.0, "OccurrenceOffsets": [20], "SurfaceForms": ["Belly Fat"]}]                      | [{"Label": "Adipose tissue", "Type": "C", "WikidataId": "Q193583", "Confidence": 1.0, "OccurrenceOffsets": [97], "SurfaceForms": ["belly fat"]}]            |
    | N86255  | health    | medical                  | Dispose of unwanted prescription drugs during the DEA's Take Back Day  |                                                                                                                                                                                                      | https://assets.msn.com/labs/mind/AAISxPN.html | [{"Label": "Drug Enforcement Administration", "Type": "O", "WikidataId": "Q622899", "Confidence": 0.992, "OccurrenceOffsets": [50], "SurfaceForms": ["DEA"]}]         | []                                                                                                                                                          |
    | N93187  | news      | newsworld                | The Cost of Trump's Aid Freeze in the Trenches of Ukraine's War        | Lt. Ivan Molchanets peeked over a parapet of sand bags at the front line of the war in Ukraine. Next to him was an empty helmet propped up to trick snipers, already perforated with multiple holes. | https://assets.msn.com/labs/mind/AAJgNsz.html | []                                                                                                                                                                    | [{"Label": "Ukraine", "Type": "G", "WikidataId": "Q212", "Confidence": 0.946, "OccurrenceOffsets": [87], "SurfaceForms": ["Ukraine"]}]                      |
    | N75236  | health    | voices                   | I Was An NBA Wife. Here's How It Affected My Mental Health.            | I felt like I was a fraud, and being an NBA wife didn't help that. In fact, it nearly destroyed me.                                                                                                  | https://assets.msn.com/labs/mind/AACk2N6.html | []                                                                                                                                                                    | [{"Label": "National Basketball Association", "Type": "O", "WikidataId": "Q155223", "Confidence": 1.0, "OccurrenceOffsets": [40], "SurfaceForms": ["NBA"]}] |
    | N99744  | health    | medical                  | How to Get Rid of Skin Tags, According to a Dermatologist              | They seem harmless, but there's a very good reason you shouldn't ignore them. The post How to Get Rid of Skin Tags, According to a Dermatologist appeared first on Reader's Digest.                  | https://assets.msn.com/labs/mind/AAAKEkt.html | [{"Label": "Skin tag", "Type": "C", "WikidataId": "Q3179593", "Confidence": 1.0, "OccurrenceOffsets": [18], "SurfaceForms": ["Skin Tags"]}]                           | [{"Label": "Skin tag", "Type": "C", "WikidataId": "Q3179593", "Confidence": 1.0, "OccurrenceOffsets": [105], "SurfaceForms": ["Skin Tags"]}]                |
    | N5771   | health    | cardio                   | Check Houston traffic map for current road conditions                  |                                                                                                                                                                                                      | https://assets.msn.com/labs/mind/AAEKnO1.html | [{"Label": "Houston", "Type": "G", "WikidataId": "Q16555", "Confidence": 0.941, "OccurrenceOffsets": [6], "SurfaceForms": ["Houston"]}]                               | []                                                                                                                                                          |
    | N124534 | sports    | football_nfl             | Should NFL be able to fine players for criticizing officiating?        | Several fines came down against NFL players for criticizing officiating this week. It's a very bad look for the league.                                                                              | https://assets.msn.com/labs/mind/AAJ4lap.html | [{"Label": "National Football League", "Type": "O", "WikidataId": "Q1215884", "Confidence": 1.0, "OccurrenceOffsets": [7], "SurfaceForms": ["NFL"]}]                  | [{"Label": "National Football League", "Type": "O", "WikidataId": "Q1215884", "Confidence": 1.0, "OccurrenceOffsets": [32], "SurfaceForms": ["NFL"]}]       |
    | N51947  | news      | newsscienceandtechnology | How to record your screen on Windows, macOS, iOS or Android            | The easiest way to record what's happening on your screen, whichever device you're using.                                                                                                            | https://assets.msn.com/labs/mind/AADlomf.html | [{"Label": "Microsoft Windows", "Type": "J", "WikidataId": "Q1406", "Confidence": 1.0, "OccurrenceOffsets": [29], "SurfaceForms": ["Windows"]}]                       | []                                                                                                                                                          |


??? note "user-sample.tsv"

    | uid | history                     |
    |-----|-----------------------------|
    | U12 | N88753,N45436,N23144        |
    | U34 | N86255,N93187,N75236,N99744 |
    | U56 | N88753,N93187,N5771         |
    | U78 | N124534,N51947,N99744       |


??? note "interaction-sample.tsv"

    | uid | nid     | click |
    |-----|---------|-------|
    | U12 | N5771   | 1     |
    | U12 | N93187  | 1     |
    | U12 | N124534 | 0     |
    | U34 | N23144  | 1     |
    | U34 | N88753  | 0     |
    | U56 | N86255  | 1     |
    | U56 | N99744  | 0     |
    | U78 | N88753  | 1     |
    | U78 | N75236  | 0     |

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

Please refer to the [Vocabulary](vocabulary.md), [Tokenizer](tokenizer.md), and [Job](job.md) pages for more information.

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

Please refer to the [UniTok](unitok.md) page for more information.

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

Please refer to the [UniTok in CLI](unitok_in_cli.md) page for more information.
