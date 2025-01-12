# Customized Tokenizer

It is common to meet the situation that the data is not suitable for the built-in tokenizers.
In this case, you can create your own tokenizer by extending the `BaseTokenizer` class.

## Case One

Assume that we have an item popularity column, each value is a float number in the range of \[0, 1\].
We would like to categorize the popularity into three levels: low, medium, and high.

```python
from unitok import DigitTokenizer


class PopularityTokenizer(DigitTokenizer):
    def __call__(self, obj: float) -> int:
        category = int(obj * 3)
        return super().__call__(category)


popularity_tokenizer = PopularityTokenizer(vocab='popularity')
print(popularity_tokenizer(0.2))  # 0
print(popularity_tokenizer(0.5))  # 1
print(popularity_tokenizer(0.8))  # 2
```

## Case Two

Assume that each item has a release date in the format of timestamp.
We would like to extract the year and month feature from the timestamp into different columns.

```python
import datetime

from unitok import BaseTokenizer, Symbol


class DateTokenizer(BaseTokenizer):
    return_list = False  # must be specified
    param_list = ['year_or_month']  # will be used when reconstructing the tokenizer
    
    YEAR = Symbol('year')
    MONTH = Symbol('month')

    def __init__(self, vocab, tokenizer_id=None, year_or_month=None, **kwargs):
        super().__init__(vocab, tokenizer_id, **kwargs)
        
        if year_or_month not in [self.YEAR.name, self.MONTH.name]:
            raise ValueError(f'Invalid year_or_month value: {year_or_month}')
        
        self.year_or_month = year_or_month

    def __call__(self, obj: float) -> int:
        date = datetime.datetime.fromtimestamp(obj)
        if self.year_or_month == self.MONTH.name:
            return super().__call__(date.month)
        return super().__call__(date.year)


year_tokenizer = DateTokenizer(vocab='year', year_or_month=DateTokenizer.YEAR.name)
month_tokenizer = DateTokenizer(vocab='month', year_or_month=DateTokenizer.MONTH.name)

current_time = datetime.datetime.now().timestamp()

print(year_tokenizer(current_time))  # 0, the first token in the year vocabulary
print(month_tokenizer(current_time))  # 0, the first token in the month vocabulary

print(year_tokenizer.vocab[0])  # '2025'
print(month_tokenizer.vocab[0])  # '1', i.e., January
```