# kathika-enig8
Code for ecom
nl2sql.py
import os
import psycopg2
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain.utilities import SQLDatabase
from langchain_experimental.sql import SQLDatabaseChain

class KSPCrimeNL2SQL:
    def __init__(self, db_uri, google_api_key):
        self.db = SQLDatabase.from_uri(db_uri)
        self.llm = ChatGoogleGenerativeAI(model="gemini-1.5-pro", google_api_key=google_api_key)
        self.chain = SQLDatabaseChain.from_llm(self.llm, self.db, verbose=True, return_direct=True)
    
    def query(self, user_input, role_filter=None):
        if role_filter:
            user_input += f" Only return data accessible to role: {role_filter}"
        try:
            result = self.chain.invoke(user_input)
            return result.get("result", "No results found")
        except Exception as e:
            return f"Error: {str(e)}"
app.py
import streamlit as st
from nl2sql import KSPCrimeNL2SQL

st.set_page_config(page_title="KSP Crime AI Assistant", layout="wide")
st.title("KSP Crime Database - Conversational AI")

db_uri = st.text_input("PostgreSQL URI", type="password")
api_key = st.text_input("Google API Key", type="password")
role = st.selectbox("Officer Role", ["Constable", "SI", "Inspector"])

if st.button("Connect") and db_uri and api_key:
    st.session_state.agent = KSPCrimeNL2SQL(db_uri, api_key)

if "agent" in st.session_state:
    query = st.text_input("Ask in English or Kannada:", placeholder="Show theft cases in Bengaluru from Jan 2025")
    if st.button("Search") and query:
        with st.spinner("Querying database..."):
            response = st.session_state.agent.query(query, role_filter=role)
            st.subheader("Result")
            st.write(response)
requirements.txt
streamlit
psycopg2-binary
langchain
langchain-experimental
langchain-google-genai
sqlalchemy
