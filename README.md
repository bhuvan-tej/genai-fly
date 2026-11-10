<div align="center">

# 🚀 GenAI Fly

**Hands-on Python notebooks for learning the building blocks of Generative AI development**

![Python](https://img.shields.io/badge/Python-3.12+-3776AB?logo=python&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)
![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)

</div>

---

## 📁 Project Structure

```
genai-fly/
├── 01-python/
│   ├── 01_Variables_Types.ipynb            # Variables, types, f-strings, collections
│   ├── 02_Conditions_Loops.ipynb           # Conditions, loops, comprehensions
│   ├── 03_Functions_DataStructures.ipynb   # Functions, lists, dicts, sets, tuples, lambdas
│   ├── 04_File_IO_JSON.ipynb               # File I/O, JSON, CSV, mini AI workflow
│   ├── 05_OOP_Classes.ipynb                # Classes, objects, methods, properties
│   └── employee.csv                        # Sample customer data read in notebook 04
├── .gitignore
└── README.md
```

## 📚 Contents

| #  | Notebook                                                | Topics                                                                                                             |
|----|---------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------|
| 01 | [Variables & Types](01-python/01_Variables_Types.ipynb) | variables · int · float · str · bool · None · type hints · type conversion · f-strings · list · dict · set · tuple |
| 02 | [Conditions & Loops](01-python/02_Conditions_Loops.ipynb) | if/elif/else · comparison, boolean & membership operators · for · range() · enumerate() · accumulation pattern · while · break/continue · list & dict comprehensions |
| 03 | [Functions & Data Structures](01-python/03_Functions_DataStructures.ipynb) | functions · parameters & return values · default parameters · \*args · \*\*kwargs · list methods · dict methods & safe access · sets · tuples · zip() · functions as objects · lambda · sorted() · map() · filter() |
| 04 | [File I/O, JSON & CSV](01-python/04_File_IO_JSON.ipynb) | json.loads() · json.dumps() · json.dump() · json.load() · `with` statement · reading & appending files · csv.DictReader · custom delimiters · CSV without headers · csv.Sniffer · mini AI workflow (prompt → data → response → log) <br> 📎 Uses [`employee.csv`](01-python/employee.csv) |
| 05 | [OOP: Classes and Objects](01-python/05_OOP_Classes.ipynb) | classes · objects · `__init__` · `self` · instance methods · `__repr__` · `@property` · working with lists of objects |

## ⚙️ Getting Started

The examples are Jupyter notebooks. To run them locally, create and activate a virtual environment, then install Jupyter:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install jupyter
jupyter notebook
```

Open a notebook from the Jupyter interface and run its cells in order. On Windows, activate the environment with `.venv\Scripts\activate`.

---

If this helped you, drop a ⭐ — it keeps the motivation going.