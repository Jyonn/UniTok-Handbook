# Integrate with PyTorch

UniTok can be easily served as a preprocessing toolkit and data loader for PyTorch models.

```python
import torch

from torch.utils.data import Dataset as BaseDataset
from unitok import UniTok

class DataSet(BaseDataset):
    def __init__(
            self,
            ut: UniTok,
    ):
        self.ut = ut

    def __getitem__(self, index):
        _sample = self.ut[index]
        sample = dict()
        for col in _sample:
            value = _sample[col]
            if self.ut.meta.jobs[col].return_list:
                 value += [0] * (self.ut.meta.jobs[col].max_len - len(_sample[col]))
            sample[col] = torch.tensor(value)
        return sample

    def __len__(self):
        return len(self.ut)

    def __iter__(self):
        for i in range(len(self)):
            yield self[i]


ut = UniTok.load('sample-ut/item')
dataset = DataSet(ut)

print(dataset[0])
# {
#   'nid': tensor(0), 
#   'category': tensor(0), 
#   'subcategory': tensor(0), 
#   'abstract@llama': tensor([ 1383,   459,   278,   ... ]), 
#   'abstract@bert': tensor([ 4497,  1996, 14960,  ...]), 
#   'title@llama': tensor([  450,  1771,  4167, ...]), 
#   'title@bert': tensor([1996, 9639, 3035, ...])
# }
```