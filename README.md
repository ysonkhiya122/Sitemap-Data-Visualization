# Sitemap Aggregation and Dashboard Documentation

## Overview
This project aggregates sitemap URLs from various news websites, processes the sitemap data, and connects it to Looker for data visualization. The resulting dashboard tracks and analyzes URL publication dates, formats, and categories for news channels.

### Websites Included:
1. ABP Live
2. AajTak
3. Jagran
4. NavBharatTimes
5. IndiaTV
6. LiveHindustan
7. Zee News
8. Republic Bharat
9. NDTV
10. TV9 Hindi
11. Times Now Hindi
12. News24

---

## Steps Involved

### 1. **Fetching Sitemap URLs from `robots.txt`**

Each website typically provides its sitemap URLs in the `robots.txt` file. We used `IMPORTXML` in Google Sheets to extract these URLs from the respective websites.

#### Formula to fetch sitemap URLs:
```excel
=IMPORTXML("https://www.abplive.com/robots.txt", "//a[contains(text(), 'sitemap')]/@href")
```

**Example of the Sitemap URL List:**
| Channel         | Sitemap URL                                                    |
|-----------------|----------------------------------------------------------------|
| ABP Live        | https://www.abplive.com/google-news-sitemap.xml                |
| AajTak          | https://www.aajtak.in/rssfeeds/news-sitemap.xml                |
| Republic Bharat | https://www.republicbharat.com/sitemap/news-sitemap-20240909.xml|

---

### 2. **Extracting Data from Sitemaps**

For each sitemap URL, we extract:
- The URLs of the individual pages.
- The publication date (`publication_date` or `lastmod`).

#### Formulas used in Google Sheets:
- **For URLs:**
   ```excel
   =IMPORTXML(Sitemap_URL!A2, "//*[local-name()='url']/*[local-name()='loc']")
   ```
- **For DateTime:**
   ```excel
   =IMPORTXML(Sitemap_URL!A2, "//*[local-name()='publication_date']") 
   OR
   =IMPORTXML(Sitemap_URL!A2, "//*[local-name()='lastmod']")
   ```

We created 12 individual sheets named after each channel (e.g., ABP, AajTak, etc.), and populated them with this extracted data.

---

### 3. **Combining Data from 12 Sheets**

Using the `QUERY` function in Google Sheets, we combined the data from all the individual channel sheets into a master sheet. This sheet contains:
- URL
- DateTime (publication or last modified date)

#### Combined `QUERY` Function:
```excel
={
  QUERY(ABP!A2:B, "Select A, B where B is not null", 0);
  QUERY(AajTak!A2:B, "Select A, B where B is not null", 0);
  QUERY(Jagran!A2:B, "Select A, B where B is not null", 0);
  QUERY(NavBharatTimes!A2:B, "Select A, B where B is not null", 0);
  QUERY(IndiaTV!A2:B, "Select A, B where B is not null", 0);
  QUERY(LiveHindustan!A2:B, "Select A, B where B is not null", 0);
  QUERY(ZeeNews!A2:B, "Select A, B where B is not null", 0);
  QUERY(RepublicBharat!A2:B, "Select A, B where B is not null", 0);
  QUERY(NDTV!A2:B, "Select A, B where B is not null", 0);
  QUERY(TV9!A2:B, "Select A, B where B is not null", 0);
  QUERY(TimesNow!A2:B, "Select A, B where B is not null", 0);
  QUERY(News24!A2:B, "Select A, B where B is not null", 0)
}
```

---

### 4. **Creating Additional Columns**

Additional columns were generated using formulas to classify the URLs by channel, format, and category.

#### 4.1 **Channel Name**
A formula was used to automatically detect the channel name based on the URL:
```excel
=IF(
  ISBLANK($A2),
  "",
  IFERROR(
    IFS(
      REGEXMATCH($A2, "ndtv"), "NDTV",
      REGEXMATCH($A2, "jagran"), "Jagran",
      REGEXMATCH($A2, "zeenews"), "Zee News",
      REGEXMATCH($A2, "abplive"), "ABP News",
      REGEXMATCH($A2, "tv9"), "TV9",
      REGEXMATCH($A2, "navbharattimes"), "NavBharat",
      REGEXMATCH($A2, "timesnow"), "Times Now",
      REGEXMATCH($A2, "indiatv"), "India TV",
      REGEXMATCH($A2, "aajtak"), "Aaj Tak",
      REGEXMATCH($A2, "news24online"), "News24",
      REGEXMATCH($A2, "republicbharat"), "R Bharat",
      REGEXMATCH($A2, "livehindustan"), "Live Hindustan"
    ),
    ""
  )
)
```

#### 4.2 **DateTime Formatting**
For consistency, the publication date was formatted uniformly:
```excel
=IF(
  ISBLANK(B2),
  "",
  IF(
    ISNUMBER(SEARCH("T", B2)),
    TEXT(INDEX(SPLIT(B2, "T"), 1), "yyyy/mm/dd") & " " & TEXT(LEFT(INDEX(SPLIT(B2, "T"), 2), 8), "H:M:S"),
    TEXT(B2, "yyyy/mm/dd H:M:S")
  )
)
```

#### 4.3 **Format (Text, Video, Photo, Web-Stories)**
To classify the format of the content:
```excel
=IF(
  ISBLANK(A2),
  "",
  IF(
    OR(
      REGEXMATCH(A2, "/video/|/short-video/|/videos/")
    ),
    "video",
    IF(
      OR(
        REGEXMATCH(A2, "/web-stories/|/web-story/")
      ),
      "web-stories",
      IF(
        REGEXMATCH(A2, "/photos/|/photo-gallery/"),
        "photo-gallery",
        "text"
      )
    )
  )
)
```

#### 4.4 **Category (News, States, Entertainment, etc.)**
Categories were dynamically extracted based on URL patterns:
```excel
=IF( OR(ISBLANK($A2), ISBLANK($E2)), "", IF( OR( INDEX(SPLIT($A2, "/"), 3) = "hindi", INDEX(SPLIT($A2, "/"), 3) = "news" ), INDEX(SPLIT($A2, "/"), 4), IF( $E2 = "text", INDEX(SPLIT($A2, "/"), 3), INDEX(SPLIT($A2, "/"), 4) ) ) )
```

---

### 5. **Connecting to Looker**

The combined data from Google Sheets was then connected to Looker for visualization and dashboard creation. The following steps were followed:

1. **Connect Google Sheets to Looker:** Import the processed data from Google Sheets into Looker using Looker's `Google Sheets` connector.
2. **Create Visualizations:** Using Looker’s data visualization tools, create charts, tables, and other visualizations to analyze URL publication patterns, format distributions, and category insights.
3. **Dashboard Creation:** A custom dashboard was built to allow for real-time analysis of:
   - Total URLs per channel.
   - Distribution of formats (text, video, web-stories, photo galleries).
   - Publication trends over time.

---

## Conclusion

This project offers a streamlined process to aggregate, process, and visualize sitemap data from various news sources. By leveraging Google Sheets and Looker, we can track and analyze the content type and publication patterns efficiently. The setup allows for continuous updating and analysis as new sitemaps are generated and published.

---

## Future Improvements
- Automating the periodic refresh of data from the sitemaps.
- Expanding the channel list to include more news sources.
- Adding more granular analysis of URL categories and subcategories.

---
