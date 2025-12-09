from fastapi import FastAPI, HTTPException, Depends, Query
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel, EmailStr, Field
from typing import Optional, List
1
from datetime import date
from sqlalchemy import create_engine, Column, Integer, String, Date,
CheckConstraint, text
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, Session
import os
DATABASE_URL = os.getenv('DATABASE_URL', 'postgresql://postgres:postgres@db:
5432/employees_db')
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()
class Employee(Base):
__tablename__ = 'employees'
id = Column(Integer, primary_key=True, index=True)
name = Column(String, nullable=False)
email = Column(String, nullable=False, unique=True, index=True)
position = Column(String, nullable=False)
department = Column(String, nullable=False)
salary = Column(Integer, nullable=False)
hire_date = Column(Date, nullable=False)
status = Column(String, nullable=False, server_default=text("'active'"))
__table_args__ = (
CheckConstraint('salary > 0', name='check_salary_positive'),
CheckConstraint("status IN ('active','inactive')",
name='check_status'),
)
# Pydantic schemas
class EmployeeCreate(BaseModel):
name: str = Field(..., min_length=1)
email: EmailStr
position: str
department: str
salary: int = Field(..., gt=0)
hire_date: date
status: Optional[str] = 'active'
class EmployeeUpdate(BaseModel):
name: Optional[str]
email: Optional[EmailStr]
position: Optional[str]
department: Optional[str]
salary: Optional[int]
hire_date: Optional[date]
status: Optional[str]
class EmployeeOut(BaseModel):
id: int
2
name: str
email: EmailStr
position: str
department: str
salary: int
hire_date: date
status: str
class Config:
orm_mode = True
app = FastAPI(title='Employee Management API')
# CORS
app.add_middleware(
CORSMiddleware,
allow_origins=["*"],
allow_credentials=True,
allow_methods=["*"],
allow_headers=["*"],
)
# Dependency
def get_db():
db = SessionLocal()
try:
yield db
finally:
db.close()
# Create tables if not exist (safe for startup)
Base.metadata.create_all(bind=engine)
# CRUD endpoints
@app.post('/api/employees', response_model=EmployeeOut, status_code=201)
def create_employee(payload: EmployeeCreate, db: Session = Depends(get_db)):
# check unique email
existing = db.query(Employee).filter(Employee.email ==
payload.email).first()
if existing:
raise HTTPException(status_code=400, detail='Email already 
registered')
emp = Employee(**payload.dict())
db.add(emp)
db.commit()
db.refresh(emp)
return emp
@app.get('/api/employees', response_model=List[EmployeeOut])
def get_employees(
db: Session = Depends(get_db),
department: Optional[str] = Query(None),
3
status: Optional[str] = Query(None),
search: Optional[str] = Query(None),
page: int = Query(1, ge=1),
page_size: int = Query(25, ge=1, le=100)
):
query = db.query(Employee)
if department:
query = query.filter(Employee.department == department)
if status:
query = query.filter(Employee.status == status)
if search:
q = f"%{search}%"
query = query.filter(
(Employee.name.ilike(q)) |
(Employee.position.ilike(q)) |
(Employee.department.ilike(q))
)
offset = (page - 1) * page_size
results =
query.order_by(Employee.id).offset(offset).limit(page_size).all()
return results
@app.get('/api/employees/{employee_id}', response_model=EmployeeOut)
def get_employee(employee_id: int, db: Session = Depends(get_db)):
emp = db.query(Employee).get(employee_id)
if not emp:
raise HTTPException(status_code=404, detail='Employee not found')
return emp
@app.put('/api/employees/{employee_id}', response_model=EmployeeOut)
def update_employee(employee_id: int, payload: EmployeeUpdate, db: Session =
Depends(get_db)):
emp = db.query(Employee).get(employee_id)
if not emp:
raise HTTPException(status_code=404, detail='Employee not found')
data = payload.dict(exclude_unset=True)
if 'email' in data:
existing = db.query(Employee).filter(Employee.email == data['email'],
Employee.id != employee_id).first()
if existing:
raise HTTPException(status_code=400, detail='Email already 
registered')
for key, value in data.items():
setattr(emp, key, value)
db.add(emp)
db.commit()
db.refresh(emp)
return emp
@app.delete('/api/employees/{employee_id}')
def delete_employee(employee_id: int, soft: bool = True, db: Session =
4
Depends(get_db)):
emp = db.query(Employee).get(employee_id)
if not emp:
raise HTTPException(status_code=404, detail='Employee not found')
if soft:
emp.status = 'inactive'
db.add(emp)
else:
db.delete(emp)
db.commit()
return {'detail': 'deleted'}
@app.get('/api/stats')
def stats(db: Session = Depends(get_db)):
total = db.query(Employee).count()
from sqlalchemy import func
by_dept = db.query(Employee.department,
func.count(Employee.id)).group_by(Employee.department).all()
avg_salary = db.query(Employee.department,
func.avg(Employee.salary)).group_by(Employee.department).all()
return {'total': total, 'by_department': dict(by_dept),
'avg_salary_by_department': {d: float(round(avg,2)) for d, avg in
avg_salary}}MM
