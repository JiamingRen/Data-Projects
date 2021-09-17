#load library
library(shiny)
library(dplyr)
library(ggplot2)
library(scales)
library(visdat)
library(naniar)
library(plotly)
library(hrbrthemes)
library(lubridate)
library(gridExtra)
library(maps)
library(ggmap)
library(leaflet)
library(Amelia)

#Q1

#load datasets
solar_data = read.csv("PE2_Solar_Data_Generation_2020_Full.csv")
solar_panel = read.csv("PE2_Solar_Panels.csv")

#check and view datasets
solar_data %>% head()
solar_panel %>% head()

solar_data %>% str()
solar_panel %>% str()

#check unique building
solar_data$MonSolarMeter %>% 
  unique() %>% 
  length()
#check unique building number
new_solar_data$BuildingNum %>% 
  unique() 

#check if two buildings in two datasets are same
setdiff(solar_panel$BuildingNum,new_solar_data$BuildingNum)

#add the MonBuildingNum to the solar data dataframe.
new_solar_data<-merge(solar_data,
                  solar_panel[,c("BuildingNum","MonBuildNum")],
                  by="BuildingNum")


#Q2


#group by building and sum energy generated in 2020
total_energy <- new_solar_data %>% 
  group_by(`MonBuildNum`) %>% 
  summarise(total = `Real.Energy.Into.the.Load..kWh.` %>%
              sum(na.rm = T))

#filter top 5 buildings that has most energy generated in 2020
top_5 <- 
  total_energy %>% arrange(desc(total)) %>% head(5)

#plot bar chart for the top 5 building
top5_bar<-top_5 %>% 
  ggplot(aes(x= reorder(MonBuildNum,total),total,fill=MonBuildNum))+
  geom_col(position = 'dodge') +
  geom_text(aes(label=total),position=position_dodge(width = 0.9),vjust=-0.2) +
  scale_y_continuous(labels = comma_format(big.mark = ".",decimal.mark = ","))+
  theme(legend.position = "top")+
  labs(x="MonSolarMeter",y="Energy Generated(KWh)")
  
top5_bar



#Q3

#1.filter top5 building all information from original dataset
#2.check NA for each columns
#3.remove NA if needed
#4.visualize how top 5 buidling energy generation varies over the course of 2020

#add the MonBuildingNum to the solar data dataframe.
new_solar_data<-merge(solar_data,
                      solar_panel[,c("BuildingNum","MonBuildNum")],
                      by="BuildingNum")


#filter top5 building all information from original dataset
fulldata_top_5 <- new_solar_data %>% filter(`MonBuildNum` %in% top_5$MonBuildNum)

#check whether building names in two datasets are consist
#setdiff(fulldata_top_5$MonSolarMeter,top_5$MonSolarMeter)

#Check NA for each columns
#plot the missing map
missmap(fulldata_top_5)

#summary missing value in a table 
fulldata_top_5 %>% 
  group_by(MonBuildNum) %>% 
  miss_var_summary()

#visualize how top 5 buildings energy generation varies over the course of 2020
plot<-fulldata_top_5 %>% 
  group_by(Timestamp =Timestamp %>% as.Date(),MonBuildNum) %>% 
  summarise(Daily_energy = sum(`Real.Energy.Into.the.Load..kWh.`,na.rm = T)) %>% 
  ggplot(aes(x=Timestamp,
             y=Daily_energy,
             group=MonBuildNum,
             colour=MonBuildNum))+
  geom_line()+
  theme(legend.position = "top",plot.title = element_text(hjust=0.5))+
  labs(x="Timestamp",y="Energy Generate(KWh)",title="Top 5 engergy generating buildings in 2020")+
  theme_ipsum()

#interactive visualization 
plot




#Q4
#create an interactive proportional symbol map using leaflet that shows the spatial positions of all 27 buildings

#get total energy generated for each building
solar_data_sum <- new_solar_data %>% 
  group_by(MonBuildNum) %>% 
  summarise(Total=Real.Energy.Into.the.Load..kWh. %>% sum(na.rm = T))

#add sums to the solar panel dataset
new_solar_panel <- left_join(solar_panel,solar_data_sum[,c("MonBuildNum","Total")],by="MonBuildNum")
#let na value equal to 0
new_solar_panel[is.na(new_solar_panel)] <-0

#a.use the provided longitude and latitude values to position the symbols
map<-new_solar_panel %>% leaflet() %>% 
  addTiles() %>% 
  #add circle to each location and the size is total / 7500
  addCircleMarkers(~solar_panel$Longitude,
             ~solar_panel$Latitude,
             popup = ~paste("Building: ",solar_panel$MonBuildNum,", ","Engergy Generated: ",Total),
             radius = ~Total/7500
             ) %>% 
  #add another marker on top of the circle because some circles are too small to see.
  addMarkers(
    ~solar_panel$Longitude,
    ~solar_panel$Latitude,
    popup = ~paste("Building: ",solar_panel$MonBuildNum,", ","Engergy Generated: ",Total),
  )




#set up ui
ui <- fluidPage(
  #row for title
  fluidRow(
    column(12,
           h1("Solar Energy Generation in Monash University Clayton Campus",
              style="font-weight:bold"),
           style="text-align:center;"
    ),
    style="border:1px solid black"
  ),
  
  #draw vis1 which shows Top 5 energy generating solar panels on buildings and the map 
  fluidRow(
    column(4,
           h3("Top 5 energy generating solar panels on buildings",
              style="font-weight:bold;"),
           p("the top 5 buildings based on their total energy generated in 2020"),
           plotOutput("vis1"),
           style="border:1px solid black"
           ),
    
    column(8,
           sliderInput(
             inputId = "slider",
             label= "Energy Generate(KWh) slider",
             min = min(new_solar_panel$Capacity..kW.),
             max= max(new_solar_panel$Capacity..kW.),
             value = range(new_solar_panel$Capacity..kW.)
           ),
           leafletOutput("map"),
           style="border:1px solid black"
           )
  ),
  
  #draw vis 2 which shows Energy generation thoughout 2020
  fluidRow(
    column(4,
           h3("Energy generation thoughout 2020",style="font-weight:bold;"),
           p("energy generation varies over the course of 2020 for the same 5 buildings"),
           style="border:1px solid black"
           ),
    column(8,
           plotlyOutput("vis2"),
           style="border:1px solid black"
           ),
    style="border:1px solid black"
    
  )
)


#Define server logic
server <- function(input, output) {
  output$vis1 <- renderPlot(
    {
      top5_bar
    }
  )
  
  output$map <- renderLeaflet(
    {
      min = input$slider[1]
      max = input$slider[2]
      new_solar_panel %>% 
        filter(Capacity..kW.>=min & `Capacity..kW.`<=max) %>% 
        leaflet() %>% 
        addTiles() %>% 
        addCircleMarkers(~Longitude,
                         ~Latitude,
                         popup = ~paste("Building: ",new_solar_panel$MonBuildNum,", ","Engergy Generated: ",Total),
                         radius = ~Total/7500
        ) %>% 
      addMarkers(
        ~Longitude,
        ~Latitude,
        popup = ~paste("Building: ",solar_panel$MonBuildNum,", ","Engergy Generated: ",Total),
      )
    }
  )
  
  output$vis2 <- renderPlotly(
    {
      plot %>% ggplotly()
    }
  )
}

#run shiny app
shinyApp(ui,server)












