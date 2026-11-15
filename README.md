# BishkekPM

**Veritas AI Hackathon 2026 · Climate track · started from the suggested project "Next-day air quality forecast"**

A next-day PM2.5 forecast for Bishkek built on measured data, turned into a bad-air warning in Kyrgyz, Russian and English, and used to test one question: **does telling the model about temperature inversions make smog warnings better?**

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/u7po00/bishkek-smog-alert/blob/main/bishkek_air_alert.ipynb)

Code: [`bishkek_air_alert.ipynb`](bishkek_air_alert.ipynb). Click the badge to open it in Google Colab, then choose Runtime → Run all. It downloads its own data, needs no GPU and finishes in a few minutes.

Demo: [live forecast and replay page](https://u7po00.github.io/bishkek-smog-alert/). Type in today's PM2.5 reading to get a forecast for the next seven days, or pick any past evening to see the warning the model would have sent and what the air turned out to be.

> This is a student prototype. It is not health advice.

## Why this project

Each winter it gets harder to simply breathe, as air pollution keeps getting worse, and international rankings place my city among the most polluted in the world. Last winter, our region had an energy crisis: the old infrastructure could not keep up with growing demand. Traffic jams are everywhere, and global warming is melting the Tien-Shan glaciers. That is life in Bishkek today. As a citizen and a patriot of my country, it hurts my soul to see all of these issues. I cannot address all these issues at once, but solving at least one of them would be better than nothing. I chose smog, as this issue is well known to every person who lives in Bishkek. Even though a forecast does not clean the air, warning the evening before lets people protect themselves.

## What I added to the suggested project

The suggested project is a next-day air quality forecast. On top of that this project adds:

1. **A local hypothesis test.** Bishkek sits in a valley and burns coal for heating. On winter nights warm air can sit on top of cold air and trap the smoke. The model is trained twice, with and without "air-trap" features (temperature in the mountains above the city minus temperature in the city, and boundary layer height where available), and the two versions are compared on the same days.
2. **A cost-aware alert.** The forecast becomes a warning. Missing a bad day counts 5 times worse than a false alarm, and the alert threshold is the one with the lowest total cost.
3. **Missed-day analysis.** A table of the bad days the alert failed to catch and how they differ from the ones it caught.
4. **An honest range.** Every forecast comes with an 80% range, and the notebook checks how often the real value fell inside it.
5. **Forecasts up to a week ahead**, to show how accuracy changes with the horizon.
6. **Warnings in Kyrgyz, Russian and English.**
7. **A demo page that runs the trained model in the browser.** It gives a live week-ahead forecast from a typed-in reading and the current weather forecast, so the dates move forward every day, and it replays every tested day with the warning in three languages.
8. **A real forecast for tomorrow.** Section 12 of the notebook takes today's PM2.5 reading, typed in by hand because the Embassy feed ended in March 2025, downloads the live weather forecast and prints tomorrow's predicted level, its range and the warning. It also prints an outlook for the next seven days. With a single reading these are less accurate than the tested results, and a month ahead cannot be forecast day by day because the weather cannot.

## Data

| What | Source |
|---|---|
| PM2.5 in Bishkek | Hourly measurements from the US Embassy monitor (AirNow format), February 2019 to June 2024, from the copy archived on Zenodo with the study "Mapping PM2.5 Sources and Emission Management Options for Bishkek, Kyrgyzstan" |
| Weather | Open-Meteo Historical Weather API (ERA5 reanalysis) for the city and for a point in the Kyrgyz Ala-Too mountains about 35 km south |

## Method

- One row per day. Inputs: PM2.5 today and over the past week (including this evening's level and today's peak), today's weather, the target day's weather (temperature, wind and its direction, humidity, rain, pressure, cloud cover, snow depth), season and weekday. Target: the target day's average PM2.5.
- Model: gradient boosting (scikit-learn `HistGradientBoostingRegressor`). It predicts how much the target day will differ from today, on a log scale.
- Baseline: "tomorrow = today".
- The target day's weather is a reanalysis, which a real forecast would not have. So the model is also trained on today's information only, and a real forecast would land between the two results.
- Testing is walk-forward. A season runs from July to June. Each season is forecast by a model trained only on earlier seasons, so every reported forecast is for a day the model had never seen. Four seasons are tested (2020/21 to 2023/24).
- A day is "bad" at 55.5 µg/m³, where the US AQI turns "Unhealthy".
- The alert threshold for each season is chosen on earlier seasons only.
- The with/without comparison is checked by resampling the test days in one-week blocks 2,000 times, which gives a 95% range for the improvement.

## Results

- **Data:** US Embassy Bishkek monitor (measured), archived on Zenodo (record 12720883); weather from Open-Meteo. 1783 days, 2019-02-06 to 2024-06-29.
- **Tested on:** 2020-07-02 to 2024-06-29, 1306 days in 4 seasons, each forecast by a model trained only on earlier seasons. 165 bad days at 55.5 ug/m3 (Unhealthy (US AQI)).
- **Accuracy (MAE, ug/m3):** naive 12.9 vs model 10.31 on all days; naive 29.0 vs model 22.59 on winter days.
- **Without the target day's weather:** model 11.04 on all days and 24.63 on winter days. A real forecast would land between this and the line above.
- **In percent:** PM2.5 averaged 31.5 ug/m3 on test days (62.8 in winter). The model's average error is 33% of that level (naive 41%). Half of the model's forecasts were within 23% of the real value (naive 25%), and 54% were within 25% of it (naive 50%).
- **Alert (thresholds 41.6 to 44.4):** caught 143 of 165 bad days with 88 false alarms (critical success index 0.57). The naive rule caught 113 with 52 false alarms (critical success index 0.52).
- **Air-trap experiment:** MAE improvement from air-trap features: +0.12 ug/m3 on all days (95% range -0.12 to +0.34); +0.24 on winter days (95% range -0.42 to +0.88). No clear difference: the range includes zero, so the gain could be chance.
- **Top drivers:** pm_today, inversion_mean_target, pm_peak_today, season_cos, pm_change.
- **Range:** the 80% range held the real value on 87% of test days.
- **Horizon (naive / model MAE):** 1 day: 12.9 / 10.31, 2 day: 17.24 / 11.77, 3 day: 19.62 / 12.78, 4 day: 20.48 / 13.17, 5 day: 20.66 / 13.71, 6 day: 21.07 / 14.32, 7 day: 21.05 / 13.99.

<!-- ILIM: 3-5 sentences in your own words. What do these numbers mean? Did the air-trap features help, and was that what you expected? What surprised you? -->

Figures are in the notebook.

## Limits

- One monitor stands in for the whole city, and it sits in a cleaner southern district. Smog in Bishkek differs a lot between neighbourhoods.
- The target day's weather is a reanalysis (what the weather turned out to be) standing in for a forecast. The results give the accuracy with it and without it; a real forecast would land in between.
- The inversion measure is a simple proxy: a mountain grid cell's temperature minus the city's, from a weather model with cells about 10 km wide.
- Winters differ a lot from each other, and there are only a few of them. Results could shift with another year.
- The measured data ends in mid-2024.
- The "cigarettes per day" figure is a rough rule of thumb used only to make the number easier to picture.

## Credits

**Data**
- US Embassy Bishkek PM2.5 monitor data, archived in [Mapping PM2.5 Sources and Emission Management Options for Bishkek, Kyrgyzstan](https://zenodo.org/records/12720883) (Zenodo record 12720883); originally published through [AirNow](https://www.airnow.gov/international/us-embassies-and-consulates/)
- [Open-Meteo](https://open-meteo.com/) Historical Weather API (CC BY 4.0), based on ERA5 reanalysis from the Copernicus Climate Change Service

**References**
- US EPA Air Quality Index categories for PM2.5, [AirNow AQI basics](https://www.airnow.gov/aqi/aqi-basics/)
- Berkeley Earth, [Air Pollution and Cigarette Equivalence](https://berkeleyearth.org/air-pollution-and-cigarette-equivalence/)

**Libraries**
- [scikit-learn](https://scikit-learn.org/), [pandas](https://pandas.pydata.org/), [NumPy](https://numpy.org/), [Matplotlib](https://matplotlib.org/), [Requests](https://requests.readthedocs.io/)

**AI tools**
- [Claude](https://claude.ai/) (Anthropic) was used to plan the project, write the notebook and demo-page code, run the notebook and draft this README. I chose the project and its additions, directed the work and wrote the "Why this project" section myself.
