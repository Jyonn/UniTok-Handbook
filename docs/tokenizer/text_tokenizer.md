# TextTokenizer Series

Text Tokenizers are designed for tokenizing a single string object, which is the most common type of data in the text processing field.
Specifically, `GloVeTokenizer` and `TransformersTokenizer` are two built-in tokenizers for text tokenization, and `BertTokenizer` is a specific instance of `TransformersTokenizer` with `key` set to `bert-base-uncased`.

## Constructor

### `CLASS` unitok.GloVeTokenizer()

| Parameter      | Default     | Type            | Example      | Description                                                                                                             |
|----------------|-------------|-----------------|--------------|-------------------------------------------------------------------------------------------------------------------------|
| `vocab`        | `N/A`       | unitok.Vocab    | `glove`      | The vocabulary to use. **It cannot be a string here. A pre-defined GloVe vocab is required.**                           |
| `tokenizer_id` | `None`      | Optional\[str\] | `'tkn-abcd'` | The ID of the tokenizer. If not provided, a random ID (auto_<id>) will be generated. It should be unique in each space. |
| `language`     | `'english'` | str             | `'english'`  | Language used for `nltk.tokenize.word_tokenize` method                                                                  |  

Please refer to [here](../faq.html#1-glove-vocabulary-construction) for GloVe vocabulary construction.

### `CLASS` unitok.TransformersTokenizer()

| Parameter      | Default | Type                       | Example               | Description                                                                                                             |
|----------------|---------|----------------------------|-----------------------|-------------------------------------------------------------------------------------------------------------------------|
| `vocab`        | `N/A`   | Union\[unitok.Vocab, str\] | `tokens`              | The vocabulary to use. If a string is provided, a new vocabulary will be created if not exists in the current space.    |
| `tokenizer_id` | `None`  | Optional\[str\]            | `'tkn-abcd'`          | The ID of the tokenizer. If not provided, a random ID (auto_<id>) will be generated. It should be unique in each space. |
| `key`          | `N/A`   | str                        | `'bert-base-uncased'` | Model key in Hugging Face                                                                                               |

### `CLASS` unitok.BertTokenizer()

| Parameter      | Default | Type                       | Example               | Description                                                                                                             |
|----------------|---------|----------------------------|-----------------------|-------------------------------------------------------------------------------------------------------------------------|
| `vocab`        | `N/A`   | Union\[unitok.Vocab, str\] | `tokens`              | The vocabulary to use. If a string is provided, a new vocabulary will be created if not exists in the current space.    |
| `tokenizer_id` | `None`  | Optional\[str\]            | `'tkn-abcd'`          | The ID of the tokenizer. If not provided, a random ID (auto_<id>) will be generated. It should be unique in each space. |

## Data Examples

> News titles and abstracts in the [MIND dataset](https://msnews.github.io/)

```json
[
  {"title": "The Brands Queen Elizabeth, Prince Charles, and Prince Philip Swear By", "abstract": "Shop the notebooks, jackets, and more that the royals can't live without."},
  {"title": "Walmart Slashes Prices on Last-Generation iPads", "abstract": "Apple's new iPad releases bring big deals on last year's models."},
  {"title": "50 Worst Habits For Belly Fat", "abstract": "hese seemingly harmless habits are holding you back and keeping you from shedding that unwanted belly fat for good."}
]
```

## Usage

```python
from unitok import GloVeTokenizer, TransformersTokenizer, BertTokenizer

glove_vocab = ...  # a pre-defined GloVe vocab, please refer to the FAQ for construction
glove_tokenizer = GloVeTokenizer(vocab=glove_vocab)
bert_tokenizer = BertTokenizer(vocab='bert')
llama1_tokenizer = TransformersTokenizer(vocab='llama', key='huggyllama/llama-7B')

print(glove_tokenizer('The Brands Queen Elizabeth, Prince Charles, and Prince Philip Swear By'))
print(bert_tokenizer('The Brands Queen Elizabeth, Prince Charles, and Prince Philip Swear By'))
print(llama1_tokenizer('The Brands Queen Elizabeth, Prince Charles, and Prince Philip Swear By'))
```