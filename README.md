# Welcome to this Totally Not Serious FIFA Outcome Prediction Tool for the 2022 World Cup

### We call it TONS OF FUN
[![CI/CD Pipeline to Docker Hub](https://github.com/andrewkroening/tons-of-fun/actions/workflows/docker_push.yml/badge.svg)](https://github.com/andrewkroening/tons-of-fun/actions/workflows/docker_push.yml)

#### Overview

This was a project for data engineering, not a statistics class. As such, the predictions are engineered to be somewhat serious, but we really used this opportunity to exercise several other things and try some new stuff. Check out the youtube demo here.

It's important to point out that this tool will work for matches outside the 2022 World Cup. If you want to simulate any matches, navigate to the [Streamlit App Landing](https://tons-of-fun.streamlit.app/) and run some simulated matches of your own!

Below is an architectural schematic of this project:

<img src="https://github.com/andrewkroening/tons-of-fun/blob/619d467cbac8841a057a47f90ce5362fc47b3e70/project_sketch.png" alt="Project Overview" width="800"/>

#### Data

We are accessing data from two locations:

* [538's Soccer Power Index](https://projects.fivethirtyeight.com/soccer-api/international/spi_global_rankings_intl.csv) - The Soccer Power Index, or SPI, is a calculated relative strength for each international team. We stream this data live each time the app is loaded and use it as the base for many of the actions in our logic tools.

* [ESPN's World Cup Scoreboard](https://www.espn.com/soccer/scoreboard?league=fifa.world&xhr=1) - JSON source data for the scoreboard web page available on ESPN. We add the `&xhr=1` to the end of the address to access the JSON, which significantly reduces the complexity of the logic required.

#### Logic

Four modules support the app:

* [`scraper.py`](https://github.com/andrewkroening/tons-of-fun/blob/c3dec07be4177a4f93a8349a6353d7ad8a01c586/logic/scraper.py) - A web scraper to get today's games from ESPN (this was easier than other scrappy/beautiful soup options we explored). We pull the JSON data behind the scoreboard web page and then query for a specific location where the score data is scored. That is transformed slightly as it is ingested into a data frame and passed to the main app.

* [`spi_dist.py`](https://github.com/andrewkroening/tons-of-fun/blob/c3dec07be4177a4f93a8349a6353d7ad8a01c586/logic/spi_dist.py) - A generator for 1000 matches between two teams. It uses the SPI dataset as its reference and calculates the SPI for each team over the 1000-match range. We introduced a variance factor of 25 to the simulations, which is pretty significant given the range of SPI values is 0-100. This is a good amount of variance because it allows for more upsets or surprises. The simulator tends to be 'less sure' than other popular prediction tools. This is also why running a simulation multiple times may result in a different result.

* [`spi_plots.py`](https://github.com/andrewkroening/tons-of-fun/blob/c3dec07be4177a4f93a8349a6353d7ad8a01c586/logic/spi_plots.py) - A plot generator that shows a layered histogram of the simulation outcomes. This tool is relatively straightforward, it just needs the outputs from `spi_dist.py`, and it will do the necessary transformations to get to the plot outcome. We used Altair for this function and are generally pleased, although Altair has a few limitations in what we could and could not customize.

* [`spi_winner`](https://github.com/andrewkroening/tons-of-fun/blob/c3dec07be4177a4f93a8349a6353d7ad8a01c586/logic/spi_winner.py) - A logic tool that returns a string describing the simulation. It uses the `spi_dist.py` output to determine which team won which percentage of the simulations and then returns the most probable winner.

#### Main App

[`streamlit_app.py`](https://github.com/andrewkroening/tons-of-fun/blob/c3dec07be4177a4f93a8349a6353d7ad8a01c586/streamlit_app.py) is the logic module that we use to bring everything together. This application file is what renders the Streamlit user interface and sets up and runs the simulations requested. Because of the nature of Streamlit's low-code requirements and the outsourcing we do to the logic modules, this app is relatively short. For example, the entire sidebar on the app landing is just a few lines of code in this file.

This file is also the one that the Streamlit Cloud service looks for when building the web app interface. It sees the accompanying [`requirements.txt`](https://github.com/andrewkroening/tons-of-fun/blob/c3dec07be4177a4f93a8349a6353d7ad8a01c586/requirements.txt) and uses that file to build a container. The Streamlit Cloud service does not need the Dockerfile we constructed; that is for a different purpose.

#### Deployment

This app is deployed through two channels. First, the Streamlit Cloud pathway:

* The [Streamlit Cloud service](https://tons-of-fun.streamlit.app) is hosting this app, which is where we recommend you use it. This free service has some limitations, but it is free, and short of taking a nap once in a while if no one visits, it will keep running in perpetuity. The service is linked to this repository and will see updates to [`streamlit_app.py`](https://github.com/andrewkroening/tons-of-fun/blob/c3dec07be4177a4f93a8349a6353d7ad8a01c586/streamlit_app.py) or [`requirements.txt`](https://github.com/andrewkroening/tons-of-fun/blob/c3dec07be4177a4f93a8349a6353d7ad8a01c586/requirements.txt) and then rebuild the container. It is able to be used by the eneral public.

As a capability exercise, we also used a more traditional deployment pipeline with the app. We do this as an example, because there may be occasions where you want to use Streamlit, but not make the app open to the public. Here's how that pathway is constructed:

* We also build a Dockerfile that is automatically pushed to Docker Hub, then on to Azure where it is sent to a Container Instance and a Web App

*We don't have public endpoints on the Azure portion enabled. It was redundant, we just wanted to see if we could do it.*

#### References

#### Credits
* Dany Jabban
* Sukhpreet Sahota
* Paul McKee
* Andrew Kroening
