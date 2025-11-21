import streamlit as st
import supabase
from datetime import datetime

# Initialize Supabase client
@st.cache_resource
def init_supabase():
    SUPABASE_URL = st.secrets["supabase_url"]
    SUPABASE_KEY = st.secrets["supabase_key"]
    client = supabase.create_client(SUPABASE_URL, SUPABASE_KEY)
    return client

# Display SQL Script in sidebar
def show_setup_instructions():
    """Display SQL setup script for manual execution"""
    with st.sidebar:
        st.header("🔧 Setup Instructions")
        
        if st.checkbox("Show SQL Setup Script"):
            st.subheader("Execute in Supabase SQL Editor:")
            
            sql_script = """-- Create fitness_classes table
CREATE TABLE fitness_classes (
  id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
  class_name TEXT NOT NULL,
  zip_code TEXT NOT NULL,
  instructor TEXT NOT NULL,
  time_slot TEXT NOT NULL,
  description TEXT,
  capacity INT NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Create index on zip_code for faster searches
CREATE INDEX idx_fitness_classes_zip_code ON fitness_classes(zip_code);

-- Enable Row Level Security
ALTER TABLE fitness_classes ENABLE ROW LEVEL SECURITY;

-- Create a policy to allow public read access
CREATE POLICY "Allow public read access" ON fitness_classes
  FOR SELECT
  USING (true);

-- Create a policy to allow public insert
CREATE POLICY "Allow public insert" ON fitness_classes
  FOR INSERT
  WITH CHECK (true);

-- Create a policy to allow public delete
CREATE POLICY "Allow public delete" ON fitness_classes
  FOR DELETE
  USING (true);"""
            
            st.code(sql_script, language="sql")
            
            st.info("""
            **To set up your database:**
            
            1. Go to your Supabase project dashboard
            2. Click **SQL Editor** → **New Query**
            3. Copy and paste the SQL script above
            4. Click **Run**
            5. Refresh this page
            """)
            
            if st.button("Copy SQL Script"):
                st.success("SQL script copied to clipboard!")

# Set page config
st.set_page_config(page_title="Fitness Classes Finder", layout="centered")
st.title("📍 Find Fitness Classes by Zip Code")

# Initialize Supabase
try:
    db = init_supabase()
except Exception as e:
    st.error("Failed to connect to Supabase. Please check your secrets.")
    st.stop()

# Show setup instructions in sidebar
show_setup_instructions()

# Create tabs for adding and viewing classes
tab1, tab2 = st.tabs(["Add Class", "View Classes"])

with tab1:
    st.subheader("Add a New Fitness Class")
    
    with st.form("class_form"):
        col1, col2 = st.columns(2)
        
        with col1:
            class_name = st.text_input("Class Name", placeholder="e.g., Yoga, CrossFit, Pilates")
            zip_code = st.text_input("Zip Code", placeholder="e.g., 10001", max_chars=5)
        
        with col2:
            instructor = st.text_input("Instructor Name", placeholder="e.g., John Doe")
            time_slot = st.selectbox("Time Slot", ["Morning (6am-9am)", "Afternoon (12pm-3pm)", "Evening (5pm-8pm)"])
        
        description = st.text_area("Description", placeholder="Brief description of the class")
        capacity = st.number_input("Class Capacity", min_value=1, max_value=100, value=20)
        
        submit_button = st.form_submit_button("Add Class")
        
        if submit_button:
            if not class_name or not zip_code or not instructor:
                st.error("Please fill in all required fields")
            elif len(zip_code) != 5 or not zip_code.isdigit():
                st.error("Please enter a valid 5-digit zip code")
            else:
                try:
                    data = {
                        "class_name": class_name,
                        "zip_code": zip_code,
                        "instructor": instructor,
                        "time_slot": time_slot,
                        "description": description,
                        "capacity": capacity,
                        "created_at": datetime.now().isoformat()
                    }
                    response = db.table("fitness_classes").insert(data).execute()
                    st.success("✅ Class added successfully!")
                except Exception as e:
                    st.error(f"Error adding class: {str(e)}")

with tab2:
    st.subheader("Search Classes by Zip Code")
    
    search_zip = st.text_input("Enter Zip Code to Search", placeholder="e.g., 10001")
    
    if search_zip:
        if len(search_zip) != 5 or not search_zip.isdigit():
            st.error("Please enter a valid 5-digit zip code")
        else:
            try:
                response = db.table("fitness_classes").select("*").eq("zip_code", search_zip).execute()
                classes = response.data
                
                if classes:
                    st.success(f"Found {len(classes)} class(es) in zip code {search_zip}")
                    for idx, cls in enumerate(classes, 1):
                        with st.container(border=True):
                            col1, col2 = st.columns([3, 1])
                            with col1:
                                st.subheading(cls["class_name"])
                                st.write(f"👨‍🏫 **Instructor:** {cls['instructor']}")
                                st.write(f"⏰ **Time:** {cls['time_slot']}")
                                st.write(f"👥 **Capacity:** {cls['capacity']} people")
                                if cls["description"]:
                                    st.write(f"📝 {cls['description']}")
                            with col2:
                                if st.button("Delete", key=f"delete_{cls['id']}", use_container_width=True):
                                    try:
                                        db.table("fitness_classes").delete().eq("id", cls["id"]).execute()
                                        st.rerun()
                                    except Exception as e:
                                        st.error(f"Error deleting class: {str(e)}")
                else:
                    st.info("No classes found in this zip code. Be the first to add one!")
            except Exception as e:
                st.error(f"Error searching classes: {str(e)}")
