# Tokenizers

The tokenizers are designed for tokenizing a group of similar objects, for example, a column of strings in the dataframe. These objects should be in same type or pattern.
UniTok provides a list of built-in tokenizers, which are all derived from the `BaseTokenizer` class.

You can also create your own tokenizer by extending the `BaseTokenizer` class. Please refer to the [Custom Tokenizer](tokenizer/custom_tokenizer.md) page for more information. 

## Built-in Tokenizers

You can click on the tokenizer name to see the details of the tokenizer.

| Tokenizer                                            | Object Example        | Return Example   | Description                                                                           |
|------------------------------------------------------|-----------------------|------------------|---------------------------------------------------------------------------------------|
| [EntityTokenizer](tokenizer/entity_tokenizer.md)     | `'apple'`             | `0`              | The object is a string, and is treated as a single token.                             |
| [EntitiesTokenizer](tokenizer/entity_tokenizer.md)   | `['apple', 'orange']` | `[0, 1]`         | The object is a list of strings, and each string is treated as a single token.        |
| [SplitTokenizer](tokenizer/split_tokenizer.md)       | `'apple orange'`      | `[0, 1]`         | The object is a string, and is split by a delimiter.                                  |
| [DigitTokenizer](tokenizer/digit_tokenizer.md)       | `123`                 | `123`            | The object is a integer, and itself will be returned.                                 |
| [DigitsTokenizer](tokenizer/digit_tokenizer.md)      | `[123, 456]`          | `[123, 456]`     | The object is a list of integers, and each integer will be returned.                  |
| [BertTokenizer](tokenizer/text_tokenizer.md)         | `'apple orange'`      | `[6207, 4589]`   | The object is a string, and is tokenized by the BertTokenizer from Hugging Face.      |
| [TransformersTokenizer](tokenizer/text_tokenizer.md) | `'apple orange'`      | `[26163, 24841]` | The object is a string, and is tokenized by a pretrained tokenizer from Hugging Face. |
| [GloVeTokenizer](tokenizer/text_tokenizer.md)        | `'apple orange'`      | `[3292, 3200]`   | The object is a string, and is tokenized by the GloVe tokenizer.                      |

## Constructor

### `CLASS` unitok.BaseTokenizer()

| Parameter      | Default | Type                       | Example      | Description                                                                                                             |
|----------------|---------|----------------------------|--------------|-------------------------------------------------------------------------------------------------------------------------|
| `vocab`        | `N/A`   | Union\[unitok.Vocab, str\] | `tokens`     | The vocabulary to use. If a string is provided, a new vocabulary will be created if not exists in the current space.    |
| `tokenizer_id` | `None`  | Optional\[str\]            | `'tkn-abcd'` | The ID of the tokenizer. If not provided, a random ID (auto_<id>) will be generated. It should be unique in each space. |

> Please carefully compare the tokenizers <`e`, `f`> with tokenizers <`h`, `i`> below. The declare order is very important. When using the vocabulary from other spaces, it will be registered in the current space. 

```python
from unitok import EntityTokenizer  # BaseTokenizer is an abstract class, we will use EntityTokenizer for demonstration
from unitok import UniTok, Vocab, VocabHub

vegetables = Vocab(name='vegetables')
music = Vocab(name='music')
tokenizer_a = EntityTokenizer(vocab='animals', tokenizer_id='entity-animals')  # OK, will use the existing vocabulary 'vegetables'

with UniTok() as ut:
    fruits = Vocab(name='fruits')
    tokenizer_b = EntityTokenizer(vocab=fruits)  # OK, will use the existing vocabulary 'fruits' with a random tokenizer ID
    tokenizer_c = EntityTokenizer(vocab='fruits', tokenizer_id='entity-fruits')  # OK, same as tokenizer_b, will use the existing vocabulary 'fruits'
    tokenizer_d = EntityTokenizer(vocab='berries', tokenizer_id='entity-fruits')  # ValueError: Conflict object declaration, the ID 'entity-fruits' is already used
    tokenizer_e = EntityTokenizer(vocab='vegetables')  # OK, create a new vocabulary named 'vegetables' with a random tokenizer ID
    tokenizer_f = EntityTokenizer(vocab=vegetables)  # ValueError: Conflict object declaration, the 'vegetables' vocabularies from the default space and the tokenizer_e are different
    tokenizer_g = EntityTokenizer(vocab='animals', tokenizer_id='entity-animals')  # OK, a new 'animals' vocabulary will be created in the ut space, and the tokenizer_id will not be conflict with the one in the default space 
    tokenizer_h = EntityTokenizer(vocab=music)  # OK, will use the existing vocabulary 'music' from the default space with a random tokenizer ID
    tokenizer_i = EntityTokenizer(vocab='music')  # OK, will use the existing vocabulary 'music' from the default space with a random tokenizer ID

with UniTok() as another_ut:
    tokenizer_j = EntityTokenizer(vocab='music')  # OK, create a new vocabulary named 'music' in the another_ut space with a random tokenizer ID
    VocabHub.add(music)  # manually add the 'music' vocabulary from the default space to the another_ut space
    tokenizer_k = EntityTokenizer(vocab='music')  # OK, will use the existing vocabulary 'music' with a random tokenizer ID
```

## Attributes

| Attribute       | Type         | Description                                           |
|-----------------|--------------|-------------------------------------------------------|
| `vocab`         | unitok.Vocab | The vocabulary used by the tokenizer.                 |
| `_tokenizer_id` | str          | The ID of the tokenizer.                              |
| `return_list`   | bool         | Whether the tokenizer should return a list of tokens. |
| `param_list`    | List\[str\]  | The list of parameters for the constructor.           |

## Magic Methods

### `METHOD` \_\_call\_\_() -> Union\[int, List\[int\]\]

Tokenize the object and return the token or a list of tokens.

| Parameter | Type | Example    | Description                        |
|-----------|------|------------|------------------------------------|
| `obj`     | Any  | `'apple'`  | The object to tokenize.            |

```python
from unitok import EntityTokenizer, EntitiesTokenizer

entity = EntityTokenizer(vocab='fruits')
print(entity('apple'))  # 0
print(entity('orange'))  # 1

entities = EntitiesTokenizer(vocab='fruits')
print(entities(['banana', 'orange']))  # [2, 1]
```

## Methods

### `METHOD` get_tokenizer_id() -> str

Return the ID of the tokenizer.

```python
from unitok import EntityTokenizer

entity = EntityTokenizer(vocab='fruits')
print(entity.get_tokenizer_id())  # 'auto_mKVTdn'
```

### `METHOD` get_classname() -> str

Return the class name of the tokenizer.

```python
from unitok import EntityTokenizer

entity = EntityTokenizer(vocab='fruits')
print(entity.get_classname())  # 'Entity'
```

### `METHOD` json() -> Dict[str, Any]

Return the metadata of the tokenizer

```python
from unitok import EntityTokenizer

entity = EntityTokenizer(vocab='fruits')
print(entity.json())  # {'tokenizer_id': 'auto_mKVTdn', 'vocab': 'fruits', 'classname': 'Entity', 'params': {}}
```