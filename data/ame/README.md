# Atomic Mass Evaluation files

Copies of the AME mass tables used by the notebooks, so that they run without downloading from the IAEA server (which often refuses requests from Colab and Kaggle).

| File | Edition | Reference |
|---|---|---|
| `mass.mas12` | AME2012 | M. Wang et al., *Chinese Physics C* **36**, 1603 (2012) |
| `mass_1.mas20.txt` | AME2020 | W. J. Huang et al., *Chinese Physics C* **45**, 030002 (2021); M. Wang et al., *Chinese Physics C* **45**, 030003 (2021) |

Source: IAEA Atomic Mass Data Center, <https://www-nds.iaea.org/amdc/>. The files are unchanged. Work that uses them should cite the AME papers above, not this repository.

AME2003 (`mass.mas03`) and AME2016 (`mass16.txt`) are not included; the v2 notebook uses them for extra rolling folds when they are placed in `ame_data/`.
