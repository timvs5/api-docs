# How to start the API

## With Docker

Run the following command in the root directory (The directory with compose.yml).

```
docker-compose -f compose.yml up
```

## Manual startup

Open command prompt and move into root folder. 

Run following command to make a virtual environment. (change venv_name to the name of your virtual environment.)
```
python -m venv venv_name
```

Run this command to start the virtual environment.
```
venv_name\Scripts\activate
```

Now run these commands to install dependencies.
```
pip install --no-cache-dir Flask==1.1.4 Flask-CORS Werkzeug MarkupSafe==1.1.1
pip install flasgger==0.9.7.1
pip install pydantic==2.6.3
pip install pandas==1.5.3
pip install seaborn==0.13.2
pip install scikit-learn==1.4.1.post1
pip install pickle-mixin
pip install redis==5.0.3
pip install pydataset==0.2.0
pip install tabulate==0.9.0
pip install psycopg2==2.9.5
pip install openpyxl==3.1.2
```

To run the api run this command.
```
python api.py
```
[More startup options](api.md#startup-flags)