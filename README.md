# UNEMPLOYMENT-ML
import folium
#Pandas to manage data
import pandas as pd
#selenium to display the html files in a browser and get screenshots
from selenium import webdriver
#time to set a delay between the opening of the browser and the screenshot
import time
#os to manage folders and path for the html, png and gif files
import os
#DivIcon to create an html div to display the year of the statistics
from folium.features import DivIcon
#imageio to turn a collection of pngs to gif
import imageio

#We turn our tsv to dataframe
df_unem_UE = pd.read_csv(os.getcwd()+"/../data/tps00203.tsv", sep='\t')
print(df_unem_UE)
#geo_UE will contain the path to the json that will create a layer to colour with the unemployment rate
geo_UE = os.getcwd()+"/../data/eu-countries.geo.json"
#This is a list that will contain the route to the png files that we will generate
pngFiles = []

#We will go year by year creating a map of the European Union map
for year in range(2007,2019):
    year = str(year)
    print("Processing "+year+":")
    #We create a map of the European Union
    ue_world = folium.Map(loc=[54.52596,15.25512],zoom_start=4,titles='Mapbox Bright',min_zoom=4,max_zoom=4)
    #With choropleth we will create a layer where we will colour the map with the diferent
    #unemployment rates of the diferent countries using the columns geo and year of our dataframe
    #geo is the code of the country in the dataframe that has to be the same as the property
    #feature.properties.sov_a3
    ue_world.choropleth(
        geo_data=geo_UE,
        data=df_unem_UE,
        columns=['geo', year],
        key_on='feature.properties.sov_a3',
        fill_color='YlOrRd', 
        fill_opacity=0.7, 
        line_opacity=0.2,
        legend_name='Unemployment',
        reset=True
        )
    #We will set the map boundaries and fix them
    ue_world.fit_bounds([[52.193636, -2.221575], [52.636878, -1.139759]])
    #We add the layer to the map
    folium.LayerControl().add_to(ue_world)
    htmlResult = os.getcwd()+"\..\htmlFiles\eu_unemp"+year+".html"
    pngResult = os.getcwd()+"\..\pngFiles\eu_unemp"+year+".png"
    #with this we create a div where we will display the year for that map
    folium.map.Marker(
    [57.193636, -11.221575],
    icon=DivIcon(
        icon_size=(150,36),
        icon_anchor=(0,0),
        html='<div style="font-size: 36pt">'+year+'</div>',
        )
    ).add_to(ue_world)
    #We save the map to the html
    ue_world.save(htmlResult)
    #We create a Chrome browser
    browser = webdriver.Chrome()
    #We open the freshly created html
    browser.get(htmlResult)
    #we maximize the window
    browser.maximize_window()
    #Give the map tiles some time to load
    time.sleep(5)
    #We take a screenshot of the browser and save it to a png file
    browser.save_screenshot(pngResult)
    #We save the file paths to the list pngFiles
    pngFiles.append(pngResult)
    #We close the browser
    browser.quit()

#images is a list where we will keep the png files opened
images = []
#We go through the screenshot and turn open them with imageio
for filename in pngFiles:
    images.append(imageio.imread(filename))
#We turn the images to our gif
imageio.mimsave('../UE_unemployment.gif', images,fps=0.5)
