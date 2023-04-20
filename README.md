# ConnorsPythonVersion
I attempted to do this in a Python/Flask enviornment

====================


### Directory Structure

url_shortener/
├── app/
│   ├── __init__.py
│   ├── models.py
│   └── views.py
└── main.py

---

### app/__init__.py

from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///urls.db'
db = SQLAlchemy(app)

---

### app/models.py

from app import db

class URL(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    original_url = db.Column(db.String, nullable=False)
    short_url = db.Column(db.String, nullable=False, unique=True)

---

### app/views.py

from app import app, db
from app.models import URL
from flask import request, redirect
import base64
import hashlib

def generate_short_url(url):
    url_hash = hashlib.sha1(url.encode()).digest()
    return base64.urlsafe_b64encode(url_hash)[:6].decode()

@app.route('/', methods=['POST'])
def shorten_url():
    original_url = request.form['url']
    short_url = generate_short_url(original_url)
    url_entry = URL(original_url=original_url, short_url=short_url)
    db.session.add(url_entry)
    db.session.commit()
    return f"Shortened URL: {short_url}"

@app.route('/<short_url>')
def redirect_url(short_url):
    url_entry = URL.query.filter_by(short_url=short_url).first()
    if url_entry:
        return redirect(url_entry.original_url)
    return "URL not found", 404

---

### main.py

from app import app, db
from app.models import URL

if __name__ == '__main__':
    db.create_all()
    app.run()

---
