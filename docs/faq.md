# FAQ

## 1. GloVe Vocabulary Construction

How to construct a GloVe vocabulary for GloVeTokenizer?

```python
import os
import subprocess
import zipfile

from unitok import Vocab


DL_URI = 'https://downloads.cs.stanford.edu/nlp/data/glove.6B.zip'
ZIP_NAME = './glove.6B.zip'
FILE_NAME = './glove.6B.300d.txt'


def get_glove_vocabulary():
    vocab = Vocab(name='glove')
    if os.path.join(vocab.filepath('.')):
        return vocab.load('.')
    
    if not os.path.exists(ZIP_NAME):
        """Download GloVe embeddings from the specified URL."""
        print(f"Downloading GloVe embeddings from {DL_URI}...")
        subprocess.run(["curl", "-o", ZIP_NAME, DL_URI], check=True)
        print(f"GloVe downloaded to {ZIP_NAME}")

    if not os.path.exists(FILE_NAME):
        """Unzip the GloVe file."""
        print(f"Unzipping GloVe embeddings...")
        with zipfile.ZipFile(ZIP_NAME, 'r') as zip_ref:
            zip_ref.extractall(FILE_NAME)
        print(f"Unzipped to {FILE_NAME}")

    """Extract unique tokens (words) from the GloVe file."""
    with open(FILE_NAME, 'r', encoding='utf-8') as f:
        for line in f:
            word, *vector = line.split()
            vocab.append(word)

    vocab.save(FILE_NAME)
```