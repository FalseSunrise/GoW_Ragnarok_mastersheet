# Web scraping project - God of War: Ragnarök equipment master sheet

## Introduction

This project aimed to create a comprehensive and easy-to-use spreadsheet with all key properties of equipment items in the game God of War: Ragnarök for creating and min-maxing in-game builds, by web-scraping data from publicly available sources using the Python programming language. 

Items chosen to include in the spreadsheet were all replaceable items for the main character, Kratos. As such, weapons have been omitted, as they are irreplaceable in game. 

Therefore, the included item types are: 
- Armor pieces (chest armor, waist armor, and wrist armor), 
- Attachments (for axe, blades, spear, and shield),
- Enchantments,
- Shields.

The spreadsheet was created to allow players to plan their item builds easier. These types of comprehensive spreadsheets are valued by game fans for easy character planning.

## Methods

### Libraries

The following Python libraries were used to build the spreadsheet:
- Requests - fetching data from URL addresses,
- BeautifulSoup - parsing HTML documents,
- Pandas - handling and transforming data, fetching and parsing tables from HTML documents, exporting
- Unidecode - handling Unicode characters.

Pandas library served as the main tool for handling fetched data using the dataframe structure and its built-in methods. For some cases, it was also possible to fetch and parse tables directly from a URL address (using html5lib and lxml libraries). 

Occasionally occurring lack of data was handled using NumPy library.

As some of the item names include Nordic letters, the Unidecode library was used for handling decoding characters from Unicode to ASCII.

### Data sources

The chosen main data source was a Fandom fan-made wiki<sup>1</sup>. It was chosen mainly due to the proven reliability of the data. An additional source, Game8 wiki<sup>2</sup>, was used for scraping perk descriptions for attachments, due to their impractical description on the Fandom wiki that deviates from the game and misses numerical values.

### Overview

Tables with armor parts, attachments, and shields on the Fandom wiki are structured similarly. Items of level 1-9 are always separated from items of levels 9, 9.1-9.3 and 10, and for armor, chest, wrist, and waist armor parts are given in separate tables (Figure 1). 
The tables themselves have a very inconvenient layout for scraping (Figure 2). In a table parsed to a dataframe, after heading, every item level occupies two rows (Figure 3). Therefore, after parsing, a dataframe must be thoroughly rearranged. 
After removing a header, the dataframe can be processed in a loop, leaping every two rows, merging two consecutive rows into one by spreading all the item properties to their target columns (Figure 4). An example of function handling splitting the data can be seen in cell [6].

![obraz](https://github.com/user-attachments/assets/e2e8f89e-d94a-4e43-9dd7-1fa3d187cd95)

Figure 1. Sample of wiki page with collapsed tables with armor set pieces.<br><br>

![image](https://github.com/user-attachments/assets/bb91554e-11c8-4e3e-813a-8e826fcba042)

Figure 2. Sample of wiki page with unwrapped table with equipment piece properties.<br><br>

![obraz](https://github.com/user-attachments/assets/f339dd71-ea7d-4adb-bdba-e1203176f004)

Figure 3. The table presented in Figure 2 parsed into a dataframe.<br><br>

![obraz](https://github.com/user-attachments/assets/4dfbee30-521c-4a15-a0a8-3b3e2257dedd)

Figure 4. The dataframe from Figure 3 after rearranging.

### Spreadsheet arrangement

The final spreadsheet should contain four tabs: Armor, Attachments, Enchantments, and Shields. Additionally, for leveled equipment (armor parts, attachments, and shields), all properties are given separately for every available item level (with level 9 being split to four sublevels: 9, 9.1, 9.2, and 9.3), whereas the enchantments can only be of the basic variant. The properties chosen to be included in the final spreadsheet, ordered, are given in Table 1.

Table 1. 

| Armor pieces                       | Attachments                           | Enchantments                       | Shields                            |
|------------------------------------|---------------------------------------|------------------------------------|------------------------------------|
| Item name                          | Item name                             | Item name                          | Item name                          |
| Set name                           | Target weapon<br>(axe, blades, spear) | Set name (if applicable)           | Item level                         |
| Part type<br>(chest, waist, wrist) | Item level                            | Statistics (if applicable)         | Statistics                         |
| Item level                         | Statistics                            | Acquisition/level <br>upgrade cost | Acquisition/level <br>upgrade cost |
| Statistics                         | Acquisition/level <br>upgrade cost    | Stat requirement (if applicable)   | Wiki link                          |
| Acquisition/level <br>upgrade cost | Wiki link                             | Perk description                   | Perk description                   |
| Wiki link                          | Perk description                      |                                    |                                    |
| Perk description                   |                                       |                                    |                                    |

## Fetching the data

### Fetching armor pieces

Armor pieces data was fetched from the Fandom wiki. Firstly, a list of armor sets with wiki links was fetched from the wiki page (https://godofwar.fandom.com/wiki/Armor_Sets_(Ragnar%C3%B6k)) using the Requests library and parsed into a BeautifulSoup object [3].

The data tables of armor parts (chest armor, wrist armor, and waist armor) for every set are spread between 6 tables across three armor parts (chest armor, wrist armor, and waist armor) and two game modes (regular and "New Game +" or "NG+"). The target tables were scraped from the armor set pages using the pandas.read_html() function [7], parsed and transformed into final format [6], the NG+ equipment level format was converted (from "9•" to 9.1, etc.) [5], and concatenated with the final dataframe "dfArmor". Finally, the missing "Spartan" set was inserted into the dataframe [8]. The obtained dataframe "dfArmor" can be seen in cell [10].

### Fetching weapon attachments

Fetching weapon attachments followed a similar approach; however, the structure of tables differs from armor pieces. Additionally, information about attachments on the Fandom wiki often misses perk information, and if present, the description is altered and misses numbers. Therefore, the perk information is fetched from a secondary source, the Game8 wiki. 

Firstly, a list of attachments with the target weapon and wiki link was fetched from the wiki page (https://godofwar.fandom.com/wiki/Attachments_(Ragnar%C3%B6k)) to a dictionary containing the target weapons as keys and a series of links to Fandom wiki pages indexed with the attachment names [11]. A similar dictionary was created for game8 wiki links [12]. The target tables with attachment data were scraped from the corresponding Fandom wiki page using the pandas.read_html() method, and simultaneously the perk information was fetched from the corresponding game8 wiki page [13]. Data was transformed using a previously used function [6] and then concatenated with the dataframe "dfAttach" [13]. This operation was performed in a loop for every attachment. Lastly, the missing perk information for "Darkdale" attachments was added separately to the final dataframe. The obtained dataframe "dfAttach" can be seen in cell [16].

### Fetching enchantments

The data about all enchantments on the Fandom wiki is gathered on a single page, spread across 62 tables:

- Tables 1-9 include grouped sets of "Realm" enchantments,
- Tables 10-33 include singular "normal" enchantments,
- Tables 34-49 include singular "Engraving" enchantments,
- Tables 50-52 include singular "Badges" enchantments, 
- Table 53 includes grouped "Burden" enchantments,
- Tables 54-62 include singular "Mystical Stones" enchantments.

Every one of these tables is laid out differently, includes different types of information, and therefore requires a different approach. However, some similarities between tables 10-52 and 54-62 make it possible to fetch them similarly.

First, enchantment tables were scraped using the pandas.read_html() method into a list of dataframes [17]. Then, dataframes 1–9 were transformed by rearranging every enchantment to a single row and splitting their properties to the relevant columns [18]. Every rearranged dataframe was concatenated with the "dfEnch" dataframe. 

The dataframe 53 was then popped to the separated variable [19] to make it possible for the remaining dataframes 10-52 and 54-62 to be processed together. Each dataframe was rearranged in a loop, splitting its properties to the relevant columns and concatenated with the "dfEnch" dataframe [20].

Lastly, the dataframe 53 containing "burdens" was rearranged, complemented with the missing "acquisition" characteristic, and concatenated with the "dfEnch" dataframe [21]. The obtained "dfEnch" dataframe can be found in cell [22].

### Fetching shields

A list with links to shield pages was created manually [23] as the Fandom wiki lacks a dedicated page containing all shield links.

Each of the shields was fetched from the corresponding page using the pandas.read_html() method [25]. After finding the correct table, the obtained dataframe was properly rearranged [24] and concatenated with the "dfShields" dataframe [25]. The procedure was similar to armor parts and weapon attachments; however, slight differences with table layouts forced writing new functions. The obtained dataframe "dfShields" can be found in cell [26].

### Exporting dataframes to .xlsx file

The last step was to export the obtained dataframes to an Excel's .xlsx file. Dataframes "dfArmor", "dfAttach", "dfEnch" and "dfShields" were exported to the "GoW_mastersheet.xlsx" file as separate sheets using pandas.ExcelWriter() method and XlsxWriter module, ommiting dataframe indices and freezing first row and first column. The obtained file can be found attached to this project.

Afterwards, the obtained file was manually formated by resizing columns and inserting filtering for columns for better clarity and usability. The finished file named "GoW_mastersheet_finished.xlsx" can be found attached to this project.

## Summary

The project is concluded with obtaining a comprehensive, versatile, and easy-to-use spreadsheet containing properties of all replaceable equipment pieces obtainable in "God of War: Ragnarök".

All leveled items can be filtered by the requested property, e.g., to compare items of the "Chest Armor" category and "Runic" attribute at a maximum level.

References
1. https://godofwar.fandom.com/wiki/God_of_War_Wiki (accessed on 23 May 2025)
2. https://game8.co/games/God-of-War-Ragnarok (accessed on 23 May 2025)
