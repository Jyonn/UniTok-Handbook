# Vocabulary

The `Vocab` class maps tokens to integers and vice versa. It also can count the occurrences of tokens during tokenization.

## Constructor

### `CLASS` unitok.Vocab()

| Parameter | Default | Type | Example    | Description                                                        | 
|-----------|---------|------|------------|--------------------------------------------------------------------|
| `name`    | `N/A`   | str  | `'tokens'` | The name of the vocabulary. It should be unique in the each space. |

```python
from unitok import Vocab, UniTok

vocab = Vocab(name='tokens')  # OK, tokens vocab in the default space
fruits = Vocab(name='fruits')  # OK, fruits vocab in the default space
another_fruits = Vocab(name='fruits')  # ValueError: Conflict object declaration

with UniTok() as ut:
    more_fruits = Vocab(name='fruits')  # OK, fruits vocab in the ut space
```

## Attributes

| Attribute   | Type                      | Description                                           |
|-------------|---------------------------|-------------------------------------------------------|
| `_name`     | str                       | The name of the vocabulary.                           |
| `_editable` | bool                      | Whether the vocabulary is editable.                   |
| `o2i`       | unitok.utils.Map          | A mapping from tokens to integers.                    |
| `i2o`       | unitok.utils.Map          | A mapping from integers to tokens.                    |
| `counter`   | unitok.vocabulary.Counter | Counts the occurrences of tokens during tokenization. |

## Properties

### `PROPERTY` name -> str

The name of the vocabulary.

```python
from unitok import Vocab

vocab = Vocab(name='tokens')
print(vocab.name)  # 'tokens'
```

### `PROPERTY` size -> int

The size of the vocabulary. Same as `__len__`.

```python 
from unitok import Vocab

vocab = Vocab(name='tokens')
print(vocab.size)  # 0
```

### `PROPERTY` editable -> bool

Whether the vocabulary is editable.

```python
from unitok import Vocab

vocab = Vocab(name='tokens')
print(vocab.editable)  # True
```

### `PROPERTY` filename -> str

The filename of the vocabulary.

```python
from unitok import Vocab

vocab = Vocab(name='tokens')
print(vocab.filename)  # 'tokens.vocab'
```

## Magic Methods

### `METHOD` \_\_iter\_\_() -> Iterator[str]

Iterates over the tokens in the vocabulary.

```python
from unitok import Vocab

vocab = Vocab(name='tokens')
vocab.extend(['apple', 'banana', 'cherry'])

for token in vocab:
    print(token)  # 'apple', 'banana', 'cherry'
```

### `METHOD` \_\_getitem\_\_() -> Union[str, int]

Returns the token by index or the index by token.

| Parameter | Default | Type | Example | Description |
|-----------|---------|------|---------|-------------|
| `item`    | `N/A`   | Union\[str, int\] | `'apple'` | The token or index. |

```python
from unitok import Vocab

vocab = Vocab(name='tokens')
vocab.extend(['apple', 'banana', 'cherry'])

print(vocab[0])  # 'apple'
print(vocab['apple'])  # 0
```

### `METHOD` \_\_contains\_\_() -> bool

Checks if a token is in the vocabulary.

| Parameter | Default | Type | Example | Description |
|-----------|---------|------|---------|-------------|
| `item`    | `N/A`   | str  | `'apple'` | The token to check. |

```python
from unitok import Vocab

vocab = Vocab(name='tokens')
vocab.extend(['apple', 'banana', 'cherry'])

print('apple' in vocab)  # True
print('orange' in vocab)  # False
```

### `METHOD` \_\_len\_\_() -> int

Returns the size of the vocabulary. Same as `size`.

```python
from unitok import Vocab

vocab = Vocab(name='tokens')
vocab.extend(['apple', 'banana', 'cherry'])

print(len(vocab))  # 3
```

## Methods

### `METHOD` append() -> int

Appends a token to the vocabulary, returning the token's index.

| Parameter   | Default | Type            | Example   | Description                                                 | 
|-------------|---------|-----------------|-----------|-------------------------------------------------------------|
| `obj`       | `N/A`   | str             | `'apple'` | The token to append.                                        | 
| `oov_token` | `None`  | Optional\[int\] | `0`       | The out-of-vocabulary token when the vocab is not editable. |

```python
from unitok import Vocab

vocab = Vocab(name='tokens')
vocab.append('apple')  # returns 0
vocab.append('banana') # returns 1

vocab.deny_edit()  # make the vocab read-only

vocab.append('apple')  # 0
vocab.append('orange')  # ValueError
vocab.append('orange', oov_token='apple')  # 0
```

### `METHOD` extend() -> List[int]

Extends the vocabulary with a list of tokens, returning the indices of the tokens.

| Parameter | Default | Type        | Example                         | Description           |
|-----------|---------|-------------|---------------------------------|-----------------------|
| `objs`    | `N/A`   | List\[str\] | `['apple', 'banana', 'cherry']` | The tokens to append. |

```python
from unitok import Vocab

vocab = Vocab(name='tokens')
vocab.extend(['apple', 'banana', 'cherry'])  # [0, 1, 2]
```

### `METHOD` deny_edit() -> self

Prevents the vocabulary from being extended.

```python
from unitok import Vocab   

vocab = Vocab(name='tokens')
vocab.deny_edit()

vocab.append('apple')  # ValueError
```

### `METHOD` allow_edit() -> self

Allows the vocabulary to be extended.

```python
from unitok import Vocab

vocab = Vocab(name='tokens')
vocab.deny_edit()

try:
    vocab.append('apple')
except ValueError:
    vocab.allow_edit()
    vocab.append('orange')

print(list(vocab))  # ['orange']
```

### `METHOD` trim() -> self

Trims the vocabulary by removing tokens with frequencies below a threshold.

| Parameter   | Default | Type | Example | Description            |
|-------------|---------|------|---------|------------------------|
| `min_count` | `N/A`   | int  | `2`     | The minimum frequency. |

```python
from unitok import Vocab

vocab = Vocab(name='tokens')

vocab.counter.activate().initialize()
vocab.extend(['apple', 'banana', 'cherry', 'apple', 'banana'])

vocab.trim(min_count=2)
print(list(vocab))  # ['apple', 'banana']
```

### `METHOD` summarize() -> Dict[tuple, int]

Summarizes the vocabulary by counting the occurrences of tokens.

| Parameter | Default | Type | Example | Description                |
|-----------|---------|------|---------|----------------------------|
| `base`    | 10      | int  | `10`    | The base of the logarithm. |

```python
from unitok import Vocab

vocab = Vocab(name='tokens')
vocab.counter.activate().initialize()

vocab.extend(['apple', 'apple', 'apple', 'apple', 'banana', 'banana', 'cherry', 'orange'])
vocab.summarize(base=2)  # {(1, 2): 2, (2, 4): 1, (4, 8): 1}

# The left bound of the key is inclusive, and the right bound is exclusive.
# (1, 2): 2, there are 2 tokens with a frequency at 1, i.e., 'cherry' and 'orange'.
# (2, 4): 1, there is 1 token with a frequency between 2 and 3, i.e., 'banana'.
# (4, 8): 1, there is 1 token with a frequency between 4 and 7, i.e., 'apple'.
```

### `METHOD` filepath() -> str

Returns the filepath of the vocabulary.

| Parameter  | Default | Type | Example | Description         |
|------------|---------|------|---------|---------------------|
| `save_dir` | `N/A`   | str  | `'.'`   | The directory path. |

```python
from unitok import Vocab

vocab = Vocab(name='tokens')
print(vocab.filepath('fruit'))  # 'fruit/tokens.vocab'
```

### `METHOD` save() -> self

Saves the vocabulary to a file.

| Parameter  | Default | Type | Example | Description         |
|------------|---------|------|---------|---------------------|
| `save_dir` | `N/A`   | str  | `'.'`   | The directory path. |

```python
from unitok import Vocab

vocab = Vocab(name='tokens')
vocab.extend(['apple', 'banana', 'cherry'])
vocab.save('fruit')
```

### `METHOD` load() -> self

Loads the vocabulary from a file.

| Parameter  | Default | Type | Example | Description         |
|------------|---------|------|---------|---------------------|
| `save_dir` | `N/A`   | str  | `'.'`   | The directory path. |

```python
from unitok import Vocab

vocab = Vocab(name='tokens').load('fruit')
print(list(vocab))  # ['apple', 'banana', 'cherry']
```

### `METHOD` json() -> dict

Returns the metadata of the vocabulary

```python
from unitok import Vocab

vocab = Vocab(name='tokens')
vocab.extend(['apple', 'banana', 'cherry'])

print(vocab.json())  # {'name': 'tokens', 'vocab_size': 3}
```
