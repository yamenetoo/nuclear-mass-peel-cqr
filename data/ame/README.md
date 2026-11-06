# Atomic Mass Evaluation files

Copies of the AME mass tables used by the notebooks, so that they run without downloading from the IAEA server (which often refuses requests from Colab and Kaggle).

| File | Edition | Reference |
|---|---|---|
| `mass.mas03` | AME2003 | G. Audi, A. H. Wapstra, C. Thibault, *Nuclear Physics A* **729**, 337 (2003) |
| `mass.mas12` | AME2012 | M. Wang et al., *Chinese Physics C* **36**, 1603 (2012) |
| `mass16.txt` | AME2016 | M. Wang et al., *Chinese Physics C* **41**, 030003 (2017) |
| `mass_1.mas20.txt` | AME2020 | W. J. Huang et al., *Chinese Physics C* **45**, 030002 (2021); M. Wang et al., *Chinese Physics C* **45**, 030003 (2021) |

Source: IAEA Atomic Mass Data Center, <https://www-nds.iaea.org/amdc/>. The files are unchanged. Work that uses them should cite the AME papers above, not this repository.

Together the four editions give the rolling temporal folds used by the notebooks (2003→2012→2016, 2012→2016→2020, 2003→2012).
