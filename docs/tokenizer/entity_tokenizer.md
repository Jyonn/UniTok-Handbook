# EntityTokenizer Series

`EntityTokenizer` and `EntitiesTokenizer` are designed for tokenizing and indexing (a list of) unsplittable objects.
`EntityTokenizer` accepts a single object and returns a single index, while `EntitiesTokenizer` accepts a list of objects and returns a list of corresponding indices. 

## Constructor

### `CLASS` unitok.EntityTokenizer()
### `CLASS` unitok.EntitiesTokenizer()

| Parameter      | Default | Type                       | Example      | Description                                                                                                             |
|----------------|---------|----------------------------|--------------|-------------------------------------------------------------------------------------------------------------------------|
| `vocab`        | `N/A`   | Union\[unitok.Vocab, str\] | `tokens`     | The vocabulary to use. If a string is provided, a new vocabulary will be created if not exists in the current space.    |
| `tokenizer_id` | `None`  | Optional\[str\]            | `'tkn-abcd'` | The ID of the tokenizer. If not provided, a random ID (auto_<id>) will be generated. It should be unique in each space. |

## Data Examples

> User age column in the [Movielens dataset](https://grouplens.org/datasets/movielens/):

```yaml
1: "Under 18"
18: "18-24"
25: "25-34"
35: "35-44"
45: "45-49"
50: "50-55"
56: "56+"
```

\* Although the dataset has already used the meaningful integers to represent each age range, these integers are not continuous and thus cannot be used as indices.

> News keywords column in the [Adressa dataset](https://reclab.idi.ntnu.no/dataset/)

```json
[
  {"keywords": ["Ferie", "Fritid", "Tr\u00f8ndelag", "Trondheim", "Skole"]},
  {"keywords": ["Tour de ski", "Langrenn"]},
  {"keywords": ["utenriks", "innenriks", "trondheim", "E6" , "midtbyen", "bybrann", "bilulykker"]}
]
```

\* The original keywords column contains a series of keyword strings concatenated with the `,` symbol, which is more suitable to apply the `SplitTokenizer`. Here is just for an illustration. 

## Usage

```python
from unitok import EntityTokenizer, Vocab

# the age index is depend on the appearance order

age_tokenizer = EntityTokenizer(vocab='age')
print(age_tokenizer('35'))  # 0
print(age_tokenizer(35))  # 0, will automatically transform to string
print(age_tokenizer('1'))  # 1
print(age_tokenizer('56'))  # 2

# or declare ages in the vocab first

age_list: list = ...
age_list = list(map(int, set(age_list)))
age_list.sort()

age_vocab = Vocab(name='ordered_age')
age_vocab.extend(age_list)  # {0: '1', 1: '18', 2: '25', 3: '35', 4: '45', 5: '50', 6: '56'}
age_vocab.deny_edit()  # assert no other age values
age_tokenizer = EntityTokenizer(vocab=age_vocab)
...
```

```python
from unitok import EntitiesTokenizer

keyword_tokenizer = EntitiesTokenizer(vocab='keyword')
print(keyword_tokenizer(["ferie", "fritid", "tr\u00f8ndelag", "trondheim", "skole"]))  # [0, 1, 2, 3, 4]
print(keyword_tokenizer(["tour de ski", "langrenn"]))  # [5, 6]
print(keyword_tokenizer(["utenriks", "innenriks", "trondheim", "E6" , "midtbyen", "bybrann", "bilulykker"]))  # [7, 8, 3, 9, 10, 11, 12]
```