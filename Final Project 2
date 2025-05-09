"""
Name: top2000_streamlit.py
CS230: Section 2
Data: Top 2000 Global Companies
URL: https://top2000-app-ahbpjnd63fycqebee8k3rk.streamlit.app/
Description:
An interactive Streamlit app to explore the Top 2000 Global Companies by filtering and visualizing data with tables, charts, and a world map.
"""

import pandas as pd
import streamlit as st
import pydeck as pdk
import matplotlib.pyplot as plt
import plotly.express as px  # [EXTRA]
import plotly.graph_objects as go  # [EXTRA]

# [DA1] Load & clean the data
@st.cache_data
def load_data():
    df = pd.read_csv("Top2000_Companies_Globally_Fixed.csv", encoding="utf-8")
    df.columns = df.columns.str.strip()
    df = df.dropna(subset=["Global Rank", "Sales ($billion)", "Profits ($billion)", "Market Value ($billion)", "Latitude_final", "Longitude_final"])
    return df

df = load_data()

# Page selector
page = st.sidebar.radio("Choose Page", ["📊 Dashboard", "🔍 Compare Companies"])

# [ST1] Continent filters
st.sidebar.header("Filter Options")
continent_list = ["All"] + sorted(df['Continent'].dropna().drop_duplicates())
selected_continent = st.sidebar.selectbox("Select a Continent", continent_list)

# [ST2] Country filter
if selected_continent == "All":
    country_options = sorted(df["Country"].dropna().drop_duplicates())
else:
    country_options = sorted(df[df["Continent"] == selected_continent]["Country"].dropna().drop_duplicates())
selected_countries = st.sidebar.multiselect("Select Country/Countries", country_options)

# [ST3] Top N
top_n = st.sidebar.slider("How many top companies to display?", min_value=5, max_value=50, value=10)

# [ST4] Market Value Range
min_mv = int(df["Market Value ($billion)"].min())
max_mv = int(df["Market Value ($billion)"].max())
value_range = st.sidebar.slider("Select Market Value Range (in $B)", min_mv, max_mv, (100, 500))

# Apply filters
filtered_df = df.copy()
if selected_continent != "All":
    filtered_df = filtered_df[filtered_df["Continent"] == selected_continent]  # [DA4]
if selected_countries:
    filtered_df = filtered_df[filtered_df["Country"].isin(selected_countries)]  # [DA5]

# [PY1]
def get_top_companies(data, n=10):
    return data.sort_values("Profits ($billion)", ascending=False).head(n)  # [DA2], [DA3]

# [PY2]
def get_market_extremes(df):
    max_val = df["Market Value ($billion)"].max()
    min_val = df["Market Value ($billion)"].min()
    return max_val, min_val

# [DA9]
high_value = filtered_df[
    (filtered_df["Market Value ($billion)"] >= value_range[0]) &
    (filtered_df["Market Value ($billion)"] <= value_range[1])
]

# Dashboard page
if page == "📊 Dashboard":
    st.title("Top 2000 Global Companies Explorer")
    st.markdown(
        "Global ranking of the top 2000 largest companies in the world based on revenue, profits, "
        "assets, and market value, as of 2020."
    )

    max_mv, min_mv = get_market_extremes(df)
    st.write(f"Max Market Value: ${max_mv}B |Min Market Value:${min_mv}B")

    # [VIZ1]
    st.subheader("High Market Value Companies")
    st.dataframe(
        high_value.sort_values("Global Rank")
        [["Global Rank", "Company", "Country", "Market Value ($billion)", "Profits ($billion)"]] #[DA7]
        .head(top_n)
    )

    # [VIZ2] Plotly Bar Chart
    st.subheader(f"Top {top_n} Most Profitable Companies (Interactive)")
    top_companies = get_top_companies(high_value, top_n)
    fig = px.bar(
        top_companies,
        x="Company",
        y="Profits ($billion)",
        title="Interactive: Top Companies by Profit",
        labels={"Profits ($billion)": "Profits ($Billion)"},
        color="Profits ($billion)",
        color_continuous_scale=["#987f97", "#addfe3"]
    )
    fig.update_layout(xaxis_tickangle=-45)
    st.plotly_chart(fig)

    # [VIZ3] Plotly Pie Chart
    st.subheader(f"Market Value Distribution (Top {top_n})")
    if not high_value.empty:
        pie_data = high_value[["Company", "Market Value ($billion)"]].sort_values("Market Value ($billion)", ascending=False).head(top_n)
        fig = px.pie(
            pie_data,
            names="Company",
            values="Market Value ($billion)",
            title="Market Value Share of Top Companies",
            color_discrete_sequence=px.colors.qualitative.Pastel1
        )
        fig.update_traces(textposition="inside", textinfo="percent+label")
        st.plotly_chart(fig)
    else:
        st.warning("No data available for the pie chart.")

    # [MAP]
    st.subheader("High Market Value Companies Map")
    continent_view = {
        "Asia": {"lat": 34, "lon": 100, "zoom": 2.5},
        "Europe": {"lat": 54, "lon": 15, "zoom": 3},
        "North America": {"lat": 40, "lon": -100, "zoom": 3},
        "South America": {"lat": -15, "lon": -60, "zoom": 3},
        "Africa": {"lat": 0, "lon": 20, "zoom": 3},
        "Oceania": {"lat": -25, "lon": 140, "zoom": 3}
    }  # [PY5]

    if selected_continent != "All" and selected_continent in continent_view:
        center = continent_view[selected_continent]
        view_state = pdk.ViewState(latitude=center["lat"], longitude=center["lon"], zoom=center["zoom"], pitch=0)
    else:
        view_state = pdk.ViewState(latitude=20, longitude=0, zoom=1.5, pitch=0)

    map_layer = pdk.Layer(
        "ScatterplotLayer",
        data=high_value,
        get_position=["Longitude_final", "Latitude_final"],
        get_radius=50000,
        get_color=[255, 0, 0],
        pickable=True
    )

    st.pydeck_chart(pdk.Deck(
        layers=[map_layer],
        initial_view_state=view_state,
        map_style="mapbox://styles/mapbox/streets-v11",
        tooltip={"text": "Company: {Company}\nCountry: {Country}\nMarket Value: {Market Value ($billion)} B"}
    ))

    # [PY3] Search bar with error handling
    try:
        user_company = st.text_input("Search for a Company by Name")
        if user_company:
            result = df[df["Company"].str.contains(user_company, case=False)]
            if not result.empty:
                st.write(result)
            else:
                st.warning("Company not found.")
    except Exception as e:
        st.error(f"Error: {e}")

# [EXTRA]Compare page - graph from plotly
elif page == "🔍 Compare Companies":
    st.title("Compare Two Companies")
    company_list = high_value["Company"].drop_duplicates().sort_values()
    company1 = st.selectbox("Select Company 1", company_list, key="comp1")
    company2 = st.selectbox("Select Company 2", company_list, key="comp2")

    if company1 and company2 and company1 != company2:
        comp_df = high_value[high_value["Company"].isin([company1, company2])].set_index("Company")
        metrics = ["Sales ($billion)", "Profits ($billion)", "Market Value ($billion)"]
        fig = go.Figure()

        fig.add_trace(go.Bar(
            x=metrics,
            y=[comp_df.loc[company1, m] for m in metrics], #[PY4]
            name=company1,
            marker_color="lightskyblue"
        ))

        fig.add_trace(go.Bar(
            x=metrics,
            y=[comp_df.loc[company2, m] for m in metrics],
            name=company2,
            marker_color="lightpink"
        ))

        fig.update_layout(
            title=f"{company1} vs {company2}",
            yaxis_title="Value ($Billion)",
            barmode="group"
        )

        st.plotly_chart(fig)
    else:
        st.info("Please select two different companies to compare.")

# [REF1] .drop_duplicates() is used to extract distinct values from a Series while preserving its structure and label information.
# This allows for consistent sorting, filtering, and integration with pandas operations.
# Reference: https://pandas.pydata.org/docs/reference/api/pandas.Series.drop_duplicates.html

# [EXTRA] Plotly Express is used for interactive visualizations.
# It is not part of CS230’s standard libraries but is widely used in real-world data applications.
# Reference: https://plotly.com/python/plotly-express/
# Colors: https://plotly.com/python/discrete-color/
