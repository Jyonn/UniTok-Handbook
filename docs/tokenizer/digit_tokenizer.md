# DigitTokenizer Series

`DigitTokenizer` and `DigitsTokenizer` are designed for those already-indexed (list of) objects.
`DigitTokenizer` accepts a single integer and returns itself, while `DigitsTokenizer` accepts a list of integers and returns themselves.

## Constructor

### `CLASS` unitok.DigitTokenizer()
### `CLASS` unitok.DigitsTokenizer()

| Parameter      | Default | Type                       | Example      | Description                                                                                                             |
|----------------|---------|----------------------------|--------------|-------------------------------------------------------------------------------------------------------------------------|
| `vocab`        | `N/A`   | Union\[unitok.Vocab, str\] | `tokens`     | The vocabulary to use. If a string is provided, a new vocabulary will be created if not exists in the current space.    |
| `tokenizer_id` | `None`  | Optional\[str\]            | `'tkn-abcd'` | The ID of the tokenizer. If not provided, a random ID (auto_<id>) will be generated. It should be unique in each space. |
| `vocab_size`   | `None`  | Optional\[int\]            | `2`          | Limits the vocabulary range into \[0, vocab_size-1\] if provided.                                                       |

## Data Examples

> Click label column in multiple recommendation datasets:

```json
[
  {"user_id": "U37448", "item_id": "I7223", "click": 0},
  {"user_id": "U95112", "item_id": "I6345", "click": 1},
  {"user_id": "U23945", "item_id": "I5496", "click": 1},
  {"user_id": "U98348", "item_id": "I1232", "click": 0},
  {"user_id": "U56502", "item_id": "I9328", "click": 1}
]
```

`user_id` and `item_id` can be tokenized by the `EntityTokenizer` class.

> User occupation column in the [Movielens dataset](https://grouplens.org/datasets/movielens/):

```yaml
 0:  "other" or not specified
 1:  "academic/educator"
 2:  "artist"
 3:  "clerical/admin"
 4:  "college/grad student"
 5:  "customer service"
 6:  "doctor/health care"
 7:  "executive/managerial"
 8:  "farmer"
 9:  "homemaker"
10:  "K-12 student"
11:  "lawyer"
12:  "programmer"
13:  "retired"
14:  "sales/marketing"
15:  "scientist"
16:  "self-employed"
17:  "technician/engineer"
18:  "tradesman/craftsman"
19:  "unemployed"
20:  "writer"
```

Unlike the user age column which needs `EntityTokenizer` to further map the integers into indices, the occupation values are continuous and can be directly serves as indices.

## Usage

```python
from unitok import DigitTokenizer

label_tokenizer = DigitTokenizer(vocab='label')
label_tokenizer(1)  # 1
label_tokenizer(0)  # 0
label_tokenizer(2)  # 2, vocabulary will be extended when error value comes

occupation_tokenizer = DigitTokenizer(vocab='occupation', vocab_size=21)
occupation_tokenizer(10)  # 10
occupation_tokenizer(5)  # 5
occupation_tokenizer(30)  # ValueError
print(len(occupation_tokenizer.vocab))  # 21, the vocabulary will be created in the constructor. Even values larger than 10 not appear, the vocabulary size is set to vocab_size.
```

```python
from unitok import DigitsTokenizer

rgb_tokenizer = DigitsTokenizer(vocab='rgb', vocab_size=256)
rgb_tokenizer([255, 255, 255])  # [255, 255, 255]
rgb_tokenizer([255, 255, 0])  # [255, 255, 0]
rgb_tokenizer([256, 255, 0])  # ValueError
```