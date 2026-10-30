import os
import sqlite3
from datetime import datetime

import numpy as np
import pandas as pd
import plotly.express as px
import streamlit as st

try:
    from openai import OpenAI
except Exception:  # pragma: no cover
    OpenAI = None

DB_PATH = "video_business_ai.db"


# --- DB setup ---
def get_conn():
    conn = sqlite3.connect(DB_PATH)
    conn.row_factory = sqlite3.Row
    return conn


def init_db():
    conn = get_conn()
    conn.execute(
        """
        CREATE TABLE IF NOT EXISTS projects (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            project_name TEXT,
            genre TEXT,
            audience TEXT,
            platform TEXT,
            format TEXT,
            status TEXT,
            budget REAL DEFAULT 0,
            expected_views REAL DEFAULT 0,
            actual_views REAL DEFAULT 0,
            revenue REAL DEFAULT 0,
            cost REAL DEFAULT 0,
            retention REAL DEFAULT 0,
            duration_days INTEGER DEFAULT 0,
            notes TEXT,
            created_at TEXT
        )
        """
    )
    conn.execute(
        """
        CREATE TABLE IF NOT EXISTS distribution_channels (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            project_id INTEGER,
            channel_name TEXT,
            channel_type TEXT,
            launch_date TEXT,
            status TEXT,
            reach REAL DEFAULT 0,
            revenue REAL DEFAULT 0,
            notes TEXT
        )
        """
    )
    conn.execute(
        """
        CREATE TABLE IF NOT EXISTS rights (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            project_name TEXT,
            rights_owner TEXT,
            rights_type TEXT,
            contract_status TEXT,
            expiry_date TEXT,
            usage_scope TEXT,
            notes TEXT
        )
        """
    )
    conn.execute(
        """
        CREATE TABLE IF NOT EXISTS tasks (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            project_id INTEGER,
            task_name TEXT,
            task_status TEXT,
            due_date TEXT,
            assignee TEXT
        )
        """
    )
    conn.commit()
    conn.close()


def seed_sample_data():
    conn = get_conn()
    count = conn.execute("SELECT COUNT(*) FROM projects").fetchone()[0]
    if count == 0:
        sample_projects = [
            {
                "project_name": "Travel Mini Docs",
                "genre": "Documentary",
                "audience": "Travelers",
                "platform": "YouTube",
                "format": "Short Form",
                "status": "Published",
                "budget": 2500,
                "expected_views": 150000,
                "actual_views": 138000,
                "revenue": 4200,
                "cost": 2300,
                "retention": 0.63,
                "duration_days": 14,
                "notes": "Strong performance on travel topics and search discovery.",
                "created_at": datetime.utcnow().strftime("%Y-%m-%d %H:%M:%S"),
            },
            {
                "project_name": "Startup Stories",
                "genre": "Branded Content",
                "audience": "Entrepreneurs",
                "platform": "LinkedIn",
                "format": "Long Form",
                "status": "Published",
                "budget": 5200,
                "expected_views": 90000,
                "actual_views": 76000,
                "revenue": 10700,
                "cost": 4900,
                "retention": 0.48,
                "duration_days": 21,
                "notes": "Good lead conversion and sponsorship potential.",
                "created_at": datetime.utcnow().strftime("%Y-%m-%d %H:%M:%S"),
            },
            {
                "project_name": "DIY Tech Tips",
                "genre": "How-To",
                "audience": "Tech Enthusiasts",
                "platform": "YouTube Shorts",
                "format": "Short Form",
                "status": "Published",
                "budget": 1200,
                "expected_views": 250000,
                "actual_views": 290000,
                "revenue": 3100,
                "cost": 1050,
                "retention": 0.71,
                "duration_days": 7,
                "notes": "Excellent retention and consistent search traffic.",
                "created_at": datetime.utcnow().strftime("%Y-%m-%d %H:%M:%S"),
            },
            {
                "project_name": "Business Interviews",
                "genre": "Interview",
                "audience": "Professionals",
                "platform": "YouTube",
                "format": "Long Form",
                "status": "Published",
                "budget": 7500,
                "expected_views": 200000,
                "actual_views": 174000,
                "revenue": 18300,
                "cost": 6600,
                "retention": 0.58,
                "duration_days": 18,
                "notes": "Strong sponsorship potential and valuable audience quality.",
                "created_at": datetime.utcnow().strftime("%Y-%m-%d %H:%M:%S"),
            },
        ]

        conn.executemany(
            """
            INSERT INTO projects (
                project_name, genre, audience, platform, format, status,
                budget, expected_views, actual_views, revenue, cost, retention,
                duration_days, notes, created_at
            ) VALUES (
                :project_name, :genre, :audience, :platform, :format, :status,
                :budget, :expected_views, :actual_views, :revenue, :cost, :retention,
                :duration_days, :notes, :created_at
            )
            """,
            sample_projects,
        )

        sample_channels = [
            (1, "YouTube", "Owned Channel", "2026-10-01", "Active", 90000, 2500, "Primary growth channel"),
            (1, "Instagram Reels", "Social", "2026-10-03", "Active", 24000, 800, "Repurposed short clips"),
            (2, "LinkedIn", "Business Channel", "2026-10-05", "Active", 19000, 4200, "Professional audience conversion"),
            (3, "YouTube Shorts", "Short Form", "2026-10-06", "Active", 118000, 1400, "Strong discovery potential"),
            (4, "YouTube", "Owned Channel", "2026-10-08", "Active", 73000, 7600, "Strong sponsor interest"),
        ]

        conn.executemany(
            """
            INSERT INTO distribution_channels (
                project_id, channel_name, channel_type, launch_date, status, reach, revenue, notes
            ) VALUES (?, ?, ?, ?, ?, ?, ?, ?)
            """,
            sample_channels,
        )

        sample_rights = [
            ("Travel Mini Docs", "Northstar Media", "License", "Active", "2027-01-15", "Global social and OTT usage", "Need annual rights check"),
            ("Startup Stories", "Core Foundry", "Sponsorship", "Active", "2026-12-31", "LinkedIn and partner site use", "Includes promotional usage"),
            ("DIY Tech Tips", "Creator Studio", "Usage", "Needs Review", "2026-11-30", "YouTube and short-form syndication", "Ensure product placement approvals"),
        ]

        conn.executemany(
            """
            INSERT INTO rights (
                project_name, rights_owner, rights_type, contract_status, expiry_date, usage_scope, notes
            ) VALUES (?, ?, ?, ?, ?, ?, ?)
            """,
            sample_rights,
        )

        sample_tasks = [
            (1, "Edit final cutdown", "Completed", "2026-10-02", "Editorial"),
            (1, "Schedule short-form reels", "In Progress", "2026-10-10", "Growth"),
            (2, "Finalize sponsor deck", "Scheduled", "2026-10-12", "Sales"),
            (3, "Publish YouTube Shorts variant", "In Progress", "2026-10-11", "Growth"),
            (4, "Review sponsorship package", "Pending", "2026-10-15", "Business Development"),
        ]

        conn.executemany(
            """
            INSERT INTO tasks (
                project_id, task_name, task_status, due_date, assignee
            ) VALUES (?, ?, ?, ?, ?)
            """,
            sample_tasks,
        )

    conn.commit()
    conn.close()


# --- Data loaders ---
def fetch_projects():
    conn = get_conn()
    df = pd.read_sql_query("SELECT * FROM projects ORDER BY created_at DESC", conn)
    conn.close()
    return df


def fetch_distribution():
    conn = get_conn()
    df = pd.read_sql_query("SELECT * FROM distribution_channels ORDER BY launch_date DESC", conn)
    conn.close()
    return df


def fetch_rights():
    conn = get_conn()
    df = pd.read_sql_query("SELECT * FROM rights ORDER BY expiry_date ASC", conn)
    conn.close()
    return df


def fetch_tasks():
    conn = get_conn()
    df = pd.read_sql_query("SELECT * FROM tasks ORDER BY due_date ASC", conn)
    conn.close()
    return df


def normalize_columns(df):
    df = df.copy()
    cols = {str(c).lower().strip(): c for c in df.columns}
    rename_map = {
        "project_name": "project_name",
        "project": "project_name",
        "title": "project_name",
        "name": "project_name",
        "genre": "genre",
        "audience": "audience",
        "platform": "platform",
        "budget": "budget",
        "expected_views": "expected_views",
        "actual_views": "actual_views",
        "revenue": "revenue",
        "cost": "cost",
        "retention": "retention",
        "status": "status",
        "format": "format",
        "duration_days": "duration_days",
        "notes": "notes",
    }
    for old_name, new_name in rename_map.items():
        if old_name in cols:
            df.rename(columns={cols[old_name]: new_name}, inplace=True)

    for col in ["project_name", "genre", "audience", "platform"]:
        if col not in df.columns:
            df[col] = ""
    for col in ["budget", "expected_views", "actual_views", "revenue", "cost", "retention", "duration_days"]:
        if col not in df.columns:
            df[col] = 0

    numeric_cols = ["budget", "expected_views", "actual_views", "revenue", "cost", "retention", "duration_days"]
    for col in numeric_cols:
        df[col] = pd.to_numeric(df[col], errors="coerce").fillna(0)

    if "status" not in df.columns:
        df["status"] = "Planning"
    if "format" not in df.columns:
        df["format"] = "Unknown"
    if "notes" not in df.columns:
        df["notes"] = ""
    if "created_at" not in df.columns:
        df["created_at"] = datetime.utcnow().strftime("%Y-%m-%d %H:%M:%S")

    return df


def insert_project_from_dataframe(df):
    conn = get_conn()
    for _, row in df.iterrows():
        conn.execute(
            """
            INSERT INTO projects (
                project_name, genre, audience, platform, format, status,
                budget, expected_views, actual_views, revenue, cost, retention,
                duration_days, notes, created_at
            ) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
            """,
            (
                str(row.get("project_name", "")),
                str(row.get("genre", "")),
                str(row.get("audience", "")),
                str(row.get("platform", "")),
                str(row.get("format", "Unknown")),
                str(row.get("status", "Planning")),
                float(row.get("budget", 0) or 0),
                float(row.get("expected_views", 0) or 0),
                float(row.get("actual_views", 0) or 0),
                float(row.get("revenue", 0) or 0),
                float(row.get("cost", 0) or 0),
                float(row.get("retention", 0) or 0),
                int(row.get("duration_days", 0) or 0),
                str(row.get("notes", "")),
                row.get("created_at", datetime.utcnow().strftime("%Y-%m-%d %H:%M:%S")),
            ),
        )
    conn.commit()
    conn.close()


def parse_uploaded_file(uploaded_file):
    name = uploaded_file.name.lower()
    if name.endswith(".csv"):
        return pd.read_csv(uploaded_file)
    if name.endswith(".xlsx") or name.endswith(".xls"):
        return pd.read_excel(uploaded_file)
    if name.endswith(".json"):
        return pd.read_json(uploaded_file)
    raise ValueError("Unsupported file type. Please upload CSV, XLSX, or JSON.")


def add_distribution_entry(project_id, channel_name, channel_type, launch_date, status, reach, revenue, notes):
    conn = get_conn()
    conn.execute(
        """
        INSERT INTO distribution_channels (
            project_id, channel_name, channel_type, launch_date, status, reach, revenue, notes
        ) VALUES (?, ?, ?, ?, ?, ?, ?, ?)
        """,
        (project_id, channel_name, channel_type, launch_date, status, reach, revenue, notes),
    )
    conn.commit()
    conn.close()


def add_rights_record(project_name, rights_owner, rights_type, contract_status, expiry_date, usage_scope, notes):
    conn = get_conn()
    conn.execute(
        """
        INSERT INTO rights (
            project_name, rights_owner, rights_type, contract_status, expiry_date, usage_scope, notes
        ) VALUES (?, ?, ?, ?, ?, ?, ?)
        """,
        (project_name, rights_owner, rights_type, contract_status, expiry_date, usage_scope, notes),
    )
    conn.commit()
    conn.close()


def add_task(project_id, task_name, task_status, due_date, assignee):
    conn = get_conn()
    conn.execute(
        """
        INSERT INTO tasks (project_id, task_name, task_status, due_date, assignee)
        VALUES (?, ?, ?, ?, ?)
        """,
        (project_id, task_name, task_status, due_date, assignee),
    )
    conn.commit()
    conn.close()


# --- Analysis ---
def calculate_project_score(row):
    score = 0.0
    score += min(float(row.get("actual_views", 0)) / 100000, 1.0) * 35
    score += min(float(row.get("revenue", 0)) / 18000, 1.0) * 30
    score += min(float(row.get("retention", 0)) / 0.70, 1.0) * 20
    score += max(0, 1 - (float(row.get("cost", 0)) / max(float(row.get("budget", 1)), 1))) * 15
    return round(score, 2)


def platform_recommendation(row):
    p = str(row.get("platform", "")).lower()
    if "youtube" in p:
        return "YouTube is strong for search growth and evergreen discovery."
    if "instagram" in p:
        return "Instagram is ideal for short-form lifestyle and creator-led engagement."
    if "linkedin" in p:
        return "LinkedIn is best for B2B and professional business storytelling."
    if "tiktok" in p:
        return "TikTok is best for trend-driven short-form virality."
    if "ott" in p:
        return "OTT is best for premium quality distribution and subscriber retention."
    return "Validate the platform against audience and content format before scaling."


def monetization_model(row):
    audience = str(row.get("audience", "")).lower()
    genre = str(row.get("genre", "")).lower()
    if "professional" in audience or "entrepreneur" in audience or "business" in audience:
        return "Sponsorships + B2B leads + premium licensing"
    if "travel" in audience or "lifestyle" in audience:
        return "Ad revenue + creator partnerships + brand integrations"
    if "tech" in audience or "creator" in audience:
        return "Ad revenue + affiliate programs + sponsor integrations"
    if "short" in str(row.get("format", "")).lower():
        return "Short-form ads + branded bundles + audience growth campaigns"
    if "documentary" in genre or "story" in genre:
        return "Festival exposure + licensing + subscription monetization"
    return "Hybrid revenue model: ads + sponsorships + licensing"


def marketing_plan(row):
    format_type = str(row.get("format", "")).lower()
    genre = str(row.get("genre", "")).lower()

    if "short" in format_type:
        return "Use a 2-second hook, 3 cutdowns, and test CTAs across channel-native versions."
    if "documentary" in genre or "story" in genre:
        return "Use teaser trailers, a release strategy around key moments, and interview-driven social clips."
    if "how-to" in genre or "educational" in genre:
        return "Build search-friendly titles, practical thumbnails, and evergreen tutorial edits."
    if "interview" in genre:
        return "Amplify using guest networks, newsletters, and community cross-posting."
    return "Pair performance creative with consistent posting and retargeting to boost conversion."


def analyze_projects(df):
    if df.empty:
        return df
    df = df.copy()
    df["score"] = df.apply(calculate_project_score, axis=1)
    df["platform_fit"] = df.apply(platform_recommendation, axis=1)
    df["monetization"] = df.apply(monetization_model, axis=1)
    df["marketing_plan"] = df.apply(marketing_plan, axis=1)
    df["priority"] = np.where(df["score"] >= 70, "High", np.where(df["score"] >= 45, "Medium", "Low"))
    return df.sort_values("score", ascending=False).reset_index(drop=True)


def generate_ai_insights(df):
    if df.empty:
        return "No data available for insights."

    if os.environ.get("OPENAI_API_KEY"):
        try:
            client = OpenAI(api_key=os.environ["OPENAI_API_KEY"])
            narrative = (
                "Analyze this video business dataset and give me a concise strategic summary with top opportunities, risks, and next actions. Data:\n"
                + df.to_json(orient="records")
            )
            response = client.chat.completions.create(
                model="gpt-4o-mini",
                messages=[
                    {"role": "system", "content": "You are a strategic AI advisor for a video development and distribution business."},
                    {"role": "user", "content": narrative},
                ],
                temperature=0.4,
            )
            return response.choices[0].message.content
        except Exception:
            pass

    scored = analyze_projects(df)
    best = scored.iloc[0] if not scored.empty else None
    avg_retention = float(df["retention"].mean()) if "retention" in df.columns else 0
    total_revenue = float(df["revenue"].sum()) if "revenue" in df.columns else 0
    total_cost = float(df["cost"].sum()) if "cost" in df.columns else 0
    avg_roi = ((total_revenue - total_cost) / max(total_cost, 1)) if total_cost else 0
    top_platform = df["platform"].mode().iloc[0] if not df.empty else "N/A"

    return "\n".join([
        "AI Strategy Summary:",
        f"- Best-performing project: {best['project_name']} with a score of {best['score']} out of 100." if best is not None else "- No project data available.",
        f"- Top platform concentration: {top_platform}.",
        f"- Average retention: {avg_retention:.1%}.",
        f"- Revenue-to-cost ratio: {avg_roi:.2f}.",
        "- Recommended actions: prioritize high-scoring projects, expand the strongest platform mix, and repurpose winners into shorter vertical formats.",
        "- Risk flags: monitor rights expiry, rising production cost, and underperforming audience-market fit on low-scoring projects.",
    ])


# --- UI rendering ---
def render_kpi_cards(df):
    if df.empty:
        st.info("No project data loaded yet.")
        return

    total_budget = float(df["budget"].sum())
    total_revenue = float(df["revenue"].sum())
    total_cost = float(df["cost"].sum())
    total_views = float(df["actual_views"].sum())
    avg_retention = float(df["retention"].mean()) if "retention" in df.columns else 0
    roi = (total_revenue - total_cost) / max(total_budget, 1)

    c1, c2, c3, c4 = st.columns(4)
    c1.metric("Total Budget", f"${total_budget:,.0f}")
    c2.metric("Total Revenue", f"${total_revenue:,.0f}")
    c3.metric("Total Views", f"{total_views:,.0f}")
    c4.metric("Avg Retention", f"{avg_retention:.1%}")

    st.markdown("### Performance snapshot")
    c5, c6, c7, c8 = st.columns(4)
    c5.metric("Total Cost", f"${total_cost:,.0f}")
    c6.metric("ROI", f"{roi:.2%}")
    c7.metric("Active Projects", str(len(df)))
    c8.metric("Avg Revenue/Project", f"${(total_revenue / max(len(df), 1)):.0f}")


def render_overview(df):
    st.subheader("Overview")
    render_kpi_cards(df)
    if df.empty:
        return

    scored = analyze_projects(df)
    platform_summary = scored.groupby("platform").agg(
        revenue=("revenue", "sum"),
        views=("actual_views", "sum"),
        projects=("project_name", "count"),
    ).reset_index()

    if not platform_summary.empty:
        chart1 = px.bar(platform_summary, x="platform", y="revenue", title="Revenue by Platform")
        chart2 = px.bar(platform_summary, x="platform", y="views", title="Views by Platform")
        c1, c2 = st.columns(2)
        c1.plotly_chart(chart1, use_container_width=True)
        c2.plotly_chart(chart2, use_container_width=True)

    st.subheader("Top opportunity list")
    st.dataframe(scored[["project_name", "platform", "genre", "score", "priority", "revenue", "actual_views", "retention"]], use_container_width=True)


def render_projects(df):
    st.subheader("Projects")
    if df.empty:
        st.info("No projects available.")
        return

    scored = analyze_projects(df)
    st.dataframe(scored[["project_name", "genre", "audience", "platform", "status", "budget", "revenue", "actual_views", "score", "priority"]], use_container_width=True)

    st.markdown("### Add project manually")
    with st.form("project_form"):
        project_name = st.text_input("Project name")
        genre = st.text_input("Genre")
        audience = st.text_input("Audience")
        platform = st.text_input("Platform")
        format_type = st.selectbox("Format", ["Short Form", "Long Form", "Series", "Podcast", "Documentary"])
        status = st.selectbox("Status", ["Planning", "In Production", "Published", "Under Review"])
        budget = st.number_input("Budget", min_value=0.0, step=100.0)
        expected_views = st.number_input("Expected views", min_value=0.0, step=1000.0)
        actual_views = st.number_input("Actual views", min_value=0.0, step=1000.0)
        revenue = st.number_input("Revenue", min_value=0.0, step=100.0)
        cost = st.number_input("Cost", min_value=0.0, step=100.0)
        retention = st.number_input("Retention (0-1)", min_value=0.0, max_value=1.0, step=0.01)
        duration_days = st.number_input("Duration days", min_value=1, step=1)
        notes = st.text_area("Notes")

        submitted = st.form_submit_button("Save project")
        if submitted and project_name:
            conn = get_conn()
            conn.execute(
                """
                INSERT INTO projects (
                    project_name, genre, audience, platform, format, status,
                    budget, expected_views, actual_views, revenue, cost, retention,
                    duration_days, notes, created_at
                ) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
                """,
                (
                    project_name,
                    genre,
                    audience,
                    platform,
                    format_type,
                    status,
                    budget,
                    expected_views,
                    actual_views,
                    revenue,
                    cost,
                    retention,
                    duration_days,
                    notes,
                    datetime.utcnow().strftime("%Y-%m-%d %H:%M:%S"),
                ),
            )
            conn.commit()
            conn.close()
            st.success(f"Project '{project_name}' saved successfully.")


def render_pipeline(df):
    st.subheader("Pipeline and tasks")
    if df.empty:
        st.info("No data available.")
        return

    tasks = fetch_tasks()
    if not tasks.empty:
        st.dataframe(tasks, use_container_width=True)

    project_list = df["project_name"].tolist()
    if project_list:
        project_choice = st.selectbox("Choose project to assign task", project_list)
        with st.form("task_form"):
            task_name = st.text_input("Task name")
            task_status = st.selectbox("Task status", ["Pending", "In Progress", "Scheduled", "Completed"])
            due_date = st.date_input("Due date")
            assignee = st.text_input("Assignee")
            submitted = st.form_submit_button("Add task")
            if submitted and task_name:
                project_id = int(df[df["project_name"] == project_choice].iloc[0]["id"]) if "id" in df.columns else None
                if project_id is not None:
                    add_task(project_id, task_name, task_status, due_date.isoformat(), assignee)
                    st.success("Task added.")
                else:
                    st.warning("Project ID not available for this row.")


def render_distribution(df):
    st.subheader("Distribution and monetization")
    projects = fetch_projects()
    if not projects.empty:
        project_choice = st.selectbox("Choose project", projects["project_name"].tolist())
        selected = projects[projects["project_name"] == project_choice].iloc[0]
        with st.form("distribution_form"):
            channel_name = st.text_input("Channel name")
            channel_type = st.selectbox("Channel type", ["Owned Channel", "Social", "OTT", "Paid", "Partner Network"])
            launch_date = st.date_input("Launch date")
            status = st.selectbox("Status", ["Scheduled", "Active", "Paused", "Completed"])
            reach = st.number_input("Reach", min_value=0.0, step=1000.0)
            revenue = st.number_input("Channel revenue", min_value=0.0, step=100.0)
            notes = st.text_area("Notes")
            submitted = st.form_submit_button("Save distribution channel")
            if submitted:
                project_id = int(selected["id"])
                add_distribution_entry(project_id, channel_name, channel_type, launch_date.isoformat(), status, reach, revenue, notes)
                st.success("Distribution entry saved.")

    dist = fetch_distribution()
    if not dist.empty:
        st.dataframe(dist, use_container_width=True)

    if not projects.empty:
        scored = analyze_projects(projects)
        st.markdown("### Monetization recommendations")
        for _, row in scored.head(10).iterrows():
            with st.expander(f"{row['project_name']}"):
                st.write("Platform fit:", row["platform_fit"])
                st.write("Monetization model:", row["monetization"])
                st.write("Marketing plan:", row["marketing_plan"])


def render_rights(df):
    st.subheader("Rights and compliance")
    with st.form("rights_form"):
        project_name = st.text_input("Project name")
        rights_owner = st.text_input("Rights owner")
        rights_type = st.selectbox("Rights type", ["License", "Usage", "Sponsorship", "Distribution", "Music Rights", "Archival Footage"])
        contract_status = st.selectbox("Contract status", ["Active", "Needs Review", "Expired", "Pending"])
        expiry_date = st.date_input("Expiry date")
        usage_scope = st.text_input("Usage scope")
        notes = st.text_area("Notes")
        submitted = st.form_submit_button("Save rights record")
        if submitted:
            add_rights_record(project_name, rights_owner, rights_type, contract_status, expiry_date.isoformat(), usage_scope, notes)
            st.success("Rights record saved.")

    rights = fetch_rights()
    if not rights.empty:
        st.dataframe(rights, use_container_width=True)


def render_ai_insights(df):
    st.subheader("AI strategy insight")
    if df.empty:
        st.info("No data available for AI analysis.")
        return

    analysis = generate_ai_insights(df)
    st.write(analysis)

    scored = analyze_projects(df)
    if not scored.empty:
        st.markdown("### Recommended project priorities")
        st.dataframe(scored[["project_name", "score", "priority", "platform_fit", "monetization", "marketing_plan"]], use_container_width=True)


def render_import_data():
    st.subheader("Import data")
    uploaded = st.file_uploader("Upload CSV, XLSX, or JSON", type=["csv", "xlsx", "xls", "json"])
    if uploaded is not None:
        try:
            df = parse_uploaded_file(uploaded)
            df = normalize_columns(df)
            if df.empty:
                st.warning("The uploaded file is empty.")
                return
            insert_project_from_dataframe(df)
            st.success(f"Imported {len(df)} records.")
        except Exception as e:
            st.error(f"Import failed: {e}")


def main():
    st.set_page_config(page_title="Advanced Video Business AI", layout="wide")
    init_db()
    seed_sample_data()

    st.title("Advanced Video Business AI")
    st.caption("Operational AI for video development, distribution, monetization, and rights management.")

    with st.sidebar:
        st.header("Controls")
        st.write("Optional: add your OpenAI API key for richer AI summaries.")
        openai_key = st.text_input("OpenAI API key", type="password", help="Optional. Leave blank to use built-in analysis.")
        if openai_key:
            os.environ["OPENAI_API_KEY"] = openai_key
        st.markdown("---")
        st.write("Database: local SQLite")
        st.write("No external setup needed to get started.")

    df = fetch_projects()
    tabs = st.tabs(["Overview", "Projects", "Pipeline", "Distribution", "Rights", "AI Insights", "Import"])

    with tabs[0]:
        render_overview(df)
    with tabs[1]:
        render_projects(df)
    with tabs[2]:
        render_pipeline(df)
    with tabs[3]:
        render_distribution(df)
    with tabs[4]:
        render_rights(df)
    with tabs[5]:
        render_ai_insights(df)
    with tabs[6]:
        render_import_data()


if __name__ == "__main__":
    main()
