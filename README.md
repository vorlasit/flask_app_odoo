# flask_app_odoo
# ✅ ขั้นตอนโดยสรุป
      ติดตั้ง Python / Flask / PostgreSQL / SQLAlchemy / Flask-Migrate
      
      สร้างโครงสร้างโปรเจกต์แบบโมดูลาร์ (แยก folder)
      
      สร้าง model ใหม่ในโฟลเดอร์ใหม่
      
      รัน flask db migrate และ flask db upgrade เพื่อสร้างตารางใน PostgreSQL

# ✅ 1. ติดตั้ง Dependency (ครั้งเดียว) 

      sudo apt update
      sudo apt install python3-pip python3-venv libpq-dev postgresql postgresql-contrib -y

  create user and password 
      
      createuser --createdb --username postgres --no-createrole --superuser --pwprompt flask_user
  or

  1. เข้าสู่ postgres shell
    
          sudo -u postgres psql
          
  2. สร้าง database  for migrate
    
          CREATE DATABASE flask_db; 
          
  3. (ถ้ายังไม่มี user ที่ตรงกับ config.py) สร้าง user และให้สิทธิ์
    
          CREATE USER flask_user WITH PASSWORD 'flask_odoo';
          
          GRANT ALL PRIVILEGES ON DATABASE flask_odoo TO flask_user;
      
  4. ออกจาก postgres
      \q

# สร้าง venv

      python3 -m venv venv
      source venv/bin/activate

# ติดตั้ง library ที่จำเป็น

      pip install flask flask_sqlalchemy flask_migrate psycopg2-binary
      
# ✅ 2. โครงสร้างโปรเจกต์ 
      flask_app/
      │
      ├── app/
      │   ├── __init__.py         ← setup app + db
      │   ├── models/
      │   │   ├── __init__.py     ← import all models
      │   │   ├── user.py         ← model User
      │   │   └── product.py      ← model Product
      │   ├── routes/
      │   │   └── main.py         ← routes
      │   └── templates/
      │
      ├── migrations/             ← auto created by flask db init
      ├── config.py               ← config DB URI
      ├── run.py                  ← entry point
# ✅ 3. config.py 

      import os 
      class Config:
          SQLALCHEMY_DATABASE_URI = 'postgresql://flask_user:flask_odoo@localhost/flask_db'
          SQLALCHEMY_TRACK_MODIFICATIONS = False
# ✅ 4. app/__init__.py 

      from flask import Flask
      from flask_sqlalchemy import SQLAlchemy
      from flask_migrate import Migrate
      from config import Config
      
      db = SQLAlchemy()
      migrate = Migrate()
      
      def create_app():
          app = Flask(__name__)
          app.config.from_object(Config)
      
          db.init_app(app)
          migrate.init_app(app, db)
      
          # Import blueprints or routes
          from .routes.main import bp as main_bp
          app.register_blueprint(main_bp)
      
          # Import models
          from .models import user, product
      
          return app
# ✅ 5. สร้าง Models ในโฟลเดอร์ app/models
app/models/user.py
 
      from .. import db
      
      class User(db.Model):
          id = db.Column(db.Integer, primary_key=True)
          email = db.Column(db.String(120), unique=True, nullable=False)
          
      app/models/product.py 
      
      from .. import db
      
      class Product(db.Model):
          id = db.Column(db.Integer, primary_key=True)
          name = db.Column(db.String(120), nullable=False)
          price = db.Column(db.Float, nullable=False)
    
app/models/__init__.py 

      from .user import User
      from .product import Product

# ✅ 6. สร้าง Route ตัวอย่าง app/routes/main.py 
      
      from flask import Blueprint, render_template
      
      bp = Blueprint('main', __name__)
      
      @bp.route('/')
      def index():
          return "Hello from Flask app!"
    
# ✅ 7. สร้าง run.py สำหรับรันแอป 

      from app import create_app
      
      app = create_app()
      
      if __name__ == '__main__':
          app.run(debug=True)
    
# ✅ 8. เริ่มต้นฐานข้อมูล (1 ครั้งเท่านั้น)
 
      export FLASK_APP=run.py
      flask db init        # ครั้งเดียว
      flask db migrate -m "Initial tables"
      flask db upgrade     # สร้าง table ใน PostgreSQL

# ✅ 9. เพิ่ม Model ใหม่ (แยกโฟลเดอร์)
คุณสามารถเพิ่ม model ใหม่ใน app/models/your_new_model.py แล้ว import ใน models/__init__.py และรัน:
       
      flask db migrate -m "Add new table"
      flask db upgrade
