### Step 1: clone github
```
# Clone your repository
git clone https://github.com/YOUR_USERNAME/house-price-analysis.git
cd house-price-analysis
```

### Step 2: Repository Structure Setup
```
house-price-analysis/
├── .gitignore
├── README.md
├── requirements.txt
├── environment.yml
├── setup.py (optional)
├── data/
│   ├── raw/
│   ├── processed/
│   └── external/
├── notebooks/
│   ├── 01-data-exploration.ipynb
│   ├── 02-data-cleaning.ipynb
│   ├── 03-feature-engineering.ipynb
│   └── 04-modeling.ipynb
├── src/
│   ├── __init__.py
│   ├── data_processing.py
│   ├── visualization.py
│   └── modeling.py
├── models/
├── reports/
│   ├── figures/
│   └── final_report.md
├── config/
│   └── config.yaml
└── docs/

```


```

# Create virtual environment
# python -m venv venv
python3.12 -m venv venv # to use a stable runtime version

# Activate virtual environment
# On Windows:
venv\Scripts\activate
# On macOS/Linux:
source venv/bin/activate

# First, upgrade pip, setuptools, and wheel
python3.12 -m pip install --upgrade pip setuptools wheel

# Install build tools
python3.12 -m pip install --upgrade build
# Set environment variables for M2 Mac
export ARCHFLAGS="-arch arm64"
export CPPFLAGS="-I/opt/homebrew/include"
export LDFLAGS="-L/opt/homebrew/lib"

# Install dependencies
pip install -r requirements.txt

# Launch Jupyter Lab
jupyter lab

```