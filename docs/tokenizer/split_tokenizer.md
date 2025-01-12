# SplitTokenizer

`SplitTokenizer` is designed for tokenizing a string that can be split into several atomic parts by a delimiter.

## Constructor

### `CLASS` unitok.SplitTokenizer()

| Parameter      | Default | Type                       | Example      | Description                                                                                                             |
|----------------|---------|----------------------------|--------------|-------------------------------------------------------------------------------------------------------------------------|
| `vocab`        | `N/A`   | Union\[unitok.Vocab, str\] | `tokens`     | The vocabulary to use. If a string is provided, a new vocabulary will be created if not exists in the current space.    |
| `tokenizer_id` | `None`  | Optional\[str\]            | `'tkn-abcd'` | The ID of the tokenizer. If not provided, a random ID (auto_<id>) will be generated. It should be unique in each space. |
| `sep`          | `N/A`   | str                        | `','`        | Delimiter that splits the string object                                                                                 |

## Data Examples

> User history in the [MIND dataset](https://msnews.github.io/)

```json
[
  {"history": "N106403 N71977 N97080 N102132 N97212 N121652"},
  {"history": "N97080 N192384 N43572 N7324 N71977"}
]
```

## Usage

```python
from unitok import SplitTokenizer

history_tokenizer = SplitTokenizer(vocab='item_id', sep=' ')
history_tokenizer("N106403 N71977 N97080 N102132 N97212 N121652")  # [0, 1, 2, 3, 4 ,5]
history_tokenizer("N97080 N192384 N43572 N7324 N71977")  # [2, 6, 7, 8, 1]
```