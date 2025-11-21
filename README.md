from langgraph.prebuilt import create_sql_agent
from langchain_community.utilities import SQLDatabase

# 1. Load your SQLite DB
db = SQLDatabase.from_uri("sqlite:///your_django_db.sqlite3")

# 2. Define your schema description in NL
schema_description = """
Here is the schema of the database you should use to reason:

Table: medical_history  
- patient_id: unique id for patient  
- blood_group: blood group of the patient  
- allergies: known allergies of the patient  
- chronic_disease: any chronic disease  
- last_visit_date: the date when patient last visited  

Table: doctor  
- doctor_id: unique id for doctor  
- doctor_name: name of the doctor  
- specialization: what the doctor specializes in  
- department: which department the doctor is part of  

Table: appointments  
- appointment_id: unique id for appointment  
- patient_id: refers to medical_history.patient_id  
- doctor_id: refers to doctor.doctor_id  
- date: date of appointment  
- status: status of the appointment (e.g., scheduled, done, canceled)  

Use only this schema **for reasoning**. When a user asks something, decide which table(s) to use, write a correct SQL query, check it, then run and give the answer.
"""

# 3. Create the LangGraph SQL Agent, but override the system messages / prompt
agent = create_sql_agent(
    db=db,
    llm=… ,  # your LLM here, e.g. ChatOpenAI
    prefix=schema_description,  # pass your schema description as prefix
    verbose=True,
)

# 4. Run a query via agent
answer = agent.run({"messages":[{"role":"user","content":"Which blood group has the most patients?"}]})
print(answer)
