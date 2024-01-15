# todo

import streamlit as st
import pandas as pd
import numpy as np
import calendar
from datetime import datetime

def delete_all_tasks():
    """Delete all tasks from the tasks DataFrame and the tasks.csv file."""
    tasks = pd.DataFrame({"Task": [], "Date": [], "Priority": [], "Status": [], "Check": []})
    tasks.to_csv("tasks.csv", index=False)
    st.experimental_rerun()

st.title("To-Do App")

# Load existing tasks
try:
    tasks = pd.read_csv("tasks.csv")
except:
    tasks = pd.DataFrame({"Task": [], "Date": [], "Priority": [], "Status": [], "Check": []})

# Remove duplicate tasks
tasks = tasks.drop_duplicates()

col1, col2, col3= st.columns(3)
# Create new task or update existing task
with col1:
    with st.form(key='task_form'):
        new_task = st.text_input("Enter new task")
        task_date = st.date_input("Select task date")
        task_priority = st.selectbox("Select task priority", ["Low", "Medium", "High"])
        task_status = st.selectbox("Select task status", ["Not Started", "In Progress", "Completed"])
        submitted = st.form_submit_button('Add Task')

    if submitted:
        existing_task = tasks.loc[(tasks["Task"] == new_task) & (tasks["Date"] == task_date) & (tasks["Priority"] == task_priority) & (tasks["Status"] == task_status)]
        if existing_task.empty:
            new_row = pd.DataFrame({"Task": [new_task], "Date": [task_date], "Priority": [task_priority], "Status": [task_status], "Check": ["False"]})
            tasks = pd.concat([tasks, new_row], ignore_index=True)
            tasks.to_csv("tasks.csv", index=False)
            st.success("Task added successfully")

# Add a Clear button at the top of the application
if st.button("Clear"):
    delete_all_tasks()
# Display existing tasks
if tasks.shape[0] > 0:
    for index, row in tasks.iterrows():
        st.checkbox(f"{row['Task']} ({row['Date']})", key=f"checkbox{index}")
else:
    st.write("No tasks to display")

with col2:
    # Update tasks in calendar
    year = datetime.now().year
    month = datetime.now().month
    calendar_html = calendar.HTMLCalendar(firstweekday=calendar.SUNDAY).formatmonth(year, month, tasks.to_dict('records'))
    st.write(calendar_html, unsafe_allow_html=True)
