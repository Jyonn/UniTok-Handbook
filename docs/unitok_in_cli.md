# UniTok in CLI

!!! note "Download the required tiny dataset for demonstration!"

    Please download UniTok-tokenized tiny dataset and unzip the file before running the code snippets. [:material-download:](https://s.6-79.cn/yYU10e/sample-ut.zip) 
    
    Please also download item parquet and try integrate action. [:material-download:](https://s.6-79.cn/i7x0uy/item.parquet)

UniTok also provide a `unitok` command line interface (CLI) for users to interact with the library in shell. 

You can use `unitok` to have an overview of the tokenized data directory, add new tokenization jobs, or remove existing jobs.
`unitok <path>` is the main command to interact with the tokenized data directory. The `<path>` argument is the path to the tokenized data directory.

## Action: summarize

To get an overview of the tokenized data directory, you can directly use `unitok <path>` without any additional arguments. It is identical to `unitok <path> --action summarize`.

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

## Action: Integrate

You can directly define a new job to tokenize the original data and integrate it into the tokenized data directory. 

| Argument             | Default | Type | Description                                                                                                      |
|----------------------|---------|------|------------------------------------------------------------------------------------------------------------------|
| `--file` / `-f`      | `N/A`   | str  | The path to the original data file, required `.parquet` format, or `.csv`, or `.tsv` format with `\t` delimiter. |
| `--column` / `-c`    | `N/A`   | str  | The column name to tokenize.                                                                                     |
| `--name` / `-n`      | `N/A`   | str  | The tokenized column name.                                                                                       |
| `--vocab` / `-v`     | `None`  | str  | The vocabulary name.                                                                                             |
| `--tokenizer` / `-t` | `None`  | str  | The tokenizer classname.                                                                                         |
| `--tokenizer_id`     | `None`  | str  | The tokenizer ID.                                                                                                |
| `--truncate`         | `None`  | int  | The maximum length of the tokenized sequence.                                                                    |
| `--lib`              | `None`  | str  | The directory path of the custom tokenizers.                                                                     |

### Example: Integrate with Built-in Tokenizers

```bash
unitok \
    sample-ut/item \
    --file item.parquet \
    --column title \
    --name title@llama2 \
    --vocab llama2 \
    --tokenizer transformers \
    --toeknizer.key meta-llama/llama-2-7b-hf \
    --tokenizer_id llama2 \
    --truncate 20 \
    --action integrate
```

```bash    
unitok \
    sample-ut/item \
    --file item.parquet \
    --column abstract \
    --name abstract@llama2 \
    --vocab llama2 \
    --tokenizer_id llama2 \  # will use the existing tokenizer  
    --truncate 50 \
    --action integrate
```

### Example: Integrate with Custom Tokenizers

> tokenizers/CounterTokenizer.py

```python
import string

from unitok import DigitTokenizer


class CounterTokenizer(DigitTokenizer):
    def __call__(self, obj: str):
        obj = obj.lower()
        count = 0
        for c in string.ascii_lowercase:
            count += c in obj
        return super().__call__(count)
```

```bash
unitok \
    sample-ut/item \
    --file item.parquet \
    --column title \
    --name title@counter \
    --vocab counter \
    --tokenizer CounterTokenizer \
    --tokenizer_id counter \
    --action integrate \
    --lib tokenizers
```

## Action: Remove

You can remove an existing job from the tokenized data directory by specifying the column name.

```bash
unitok \
    sample-ut/item \
    --name title@bert \
    --action remove
```
