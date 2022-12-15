# CS213final
import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt

# 1 init dataframe from csv file
district_df = pd.read_csv('BostonPoliceDistricts.csv')
df = pd.read_csv('BostonCrime2021_7000_sample.csv')
table = df[['OFFENSE_CODE', 'OFFENSE_DESCRIPTION', 'DISTRICT', 'YEAR', 'MONTH', 'DAY_OF_WEEK', 'STREET',
            'Lat', 'Long']]

# display search condition
with st.sidebar:
    st.title("Search condition")

    search_offense_desc = st.text_input('Offense description:', '')

    search_month = st.selectbox(
        'Month of occurrence:',
        [None, 1,2,3,4,5,6,7,8,9,10,11,12])

    district_df.loc[0]=['all district', 'all district']
    choose_district_name = st.radio('District of occurrence:', district_df['District Name'])
    choose_district = district_df.loc[district_df['District Name'] == choose_district_name].iloc[0,0]


# display data content and charts
st.title("Boston Crime")
'This page is used to display Boston crime statistics, including one table and two charts.'

st.header("Table information")

match1 = table['OFFENSE_DESCRIPTION'].str.contains(search_offense_desc.upper())
if search_month is not None:
    match2 = table['MONTH'] == search_month
else:
    match2 = True

if choose_district == 'all district':
    match3 = True
else:
    match3 = table['DISTRICT'] == choose_district

filter_table = table.loc[match1 & match2 & match3]
filter_table

st.header("Pie chart")
pie_df = filter_table[['DISTRICT','OFFENSE_CODE']].groupby('DISTRICT', as_index=False).count()
pie_result = pd.merge(pie_df, district_df, left_on="DISTRICT", right_on="District")

fig, axes = plt.subplots()
axes.pie(x=pie_result['OFFENSE_CODE'], labels = pie_result['District Name'], autopct='%1.1f%%')
st.pyplot(fig)

# line
st.header("Line chart")

chart_data = filter_table[['DAY_OF_WEEK', 'OFFENSE_CODE']]

line = chart_data.groupby('DAY_OF_WEEK').count()
line.rename(columns={'OFFENSE_CODE': 'Quantity'}, inplace=True)
st.line_chart(line)

# map
st.header("Map")
map_data = filter_table[['Lat','Long']].rename(columns={'Lat':'lat', 'Long':'lon'})
st.map(map_data)
