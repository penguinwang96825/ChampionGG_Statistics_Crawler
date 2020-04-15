# Champion Statistics Crawler
A note to build a crawler to scrape heros statistics data from [here](https://champion.gg/).

## Import packages
```python
import pandas as pd
import numpy as np
import requests
import bs4
from bs4 import BeautifulSoup
from tqdm.notebook import tqdm
```
## Main Code
### Get all URLs
```python
def get_heros_urls():  
    # Create an empty list to store heros statistics data
    hero_summary = []
    # Get html by using requests
    url = "https://champion.gg/"
    r = requests.get(url, timeout=30).text
    # Use BeautifulSoup to parse the content
    soup = BeautifulSoup(r, "html.parser")
    # Find the heros information
    table = soup.find(name="div", attrs={"class": "col-md-9 clearfix"})
    for hero in table.find_all(name="div", attrs={"class": "champ-height"}):  
        # Create an empty list to get hero name temporaryly
        s = []
        name = hero.find(name="span", attrs={"class": "champion-name"}).string
        # 3 counterexamples: (Nunu & Willump, Jungle), (Dr. Mundo, Top), (Dr. Mundo, Jungle)
        name = name.split("&")[0]
        name = name.replace(" ", "").replace(".", "")
        s.append(name)
        # Get lane of the hero
        for lane in hero.find_all(name="a")[1:]:
            l = lane.string
            l = l.replace(" ", "").replace("\n", "")
            s.append(l)
        hero_summary.append(s)
    # Create an empty list to store url
    urls = []
    basic_url = "https://champion.gg/champion/"
    for i in range(len(hero_summary)):
        for lane in hero_summary[i][1:]:
            url = basic_url + str(hero_summary[i][0]) + "/" + str(lane)
            urls.append(url)
    return urls
```

### Get statistics data
```python
def get_heros_statistics(url):
    # Get hero name and lane from string of url
    hero_name = url.split("/")[-2]
    lane = url.split("/")[-1]
    hero_stat = []
    hero_stat.append(hero_name)
    hero_stat.append(lane)

    r = requests.get(url, timeout=30).text
    soup = BeautifulSoup(r, "html.parser")
    tbody = soup.find(name="tbody")
    if tbody is not None:
        for tr in tbody.find_all(name="tr", limit=11):
            # Find average score in every rows
            tds = tr.find_all(name="td")
            if tds[1] is not None:
                average = tds[1].string
                average = str(average).replace(" ", "").replace("\n", "")
            hero_stat.append(average)

            # Find role placement ratio in every rows
            tds = tr.find_all(name="td")
            if tds[2].find(name="strong") is not None:
                role_placement_1st = tds[2].find(name="strong").string
                role_placement_1st = str(role_placement_1st).replace(" ", "").replace("\n", "")
            if tds[2].find(name="small") is not None: 
                role_placement_2nd = tds[2].find(name="small").string
                role_placement_2nd = str(role_placement_2nd).replace(" ", "").replace("/", "")
            role_placement = role_placement_1st + "/" + role_placement_2nd
            hero_stat.append(role_placement)
        
        # Find overall placement ratio
        tr = tbody.find_all(name="tr")
        overall_placement_1st = tr[-1].find(name="strong").string.replace(" ", "").replace("\n", "")
        overall_placement_2nd = tr[-1].find(name="small").string.replace(" ", "").replace("/", "")
        overall_placement = overall_placement_1st + "/" + overall_placement_2nd
        hero_stat.append(overall_placement)
    
    return hero_stat
```

### Get heros statistics dataframe
```python
def get_heros_dataframe():
    stat = []
    urls = get_heros_urls()
    for url in tqdm(urls):
        hero_stat = get_heros_statistics(url)
        stat.append(hero_stat)
        df = pd.DataFrame(stat)
    df.columns = ["hero", "lane", 
       "win_rate_average", "win_rate_rp", 
       "play_rate_average", "play_rate_rp", 
       "ban_rate_average", "ban_rate_rp", 
       "player_base_average_games_played_average", "player_base_average_games_played_rp", 
       "gold_earned_average", "gold_earned_rp", 
       "kills_average", "kills_rp", 
       "deaths_average", "death_rp", 
       "assists_average", "assists_rp", 
       "damage_dealt_average", "damage_dealt_rp", 
       "damage_taken_average", "amage_taken_rp", 
       "minions_killed_average", "minions_killed_rp", 
       "overall_placement_rp"]
    return df
```

