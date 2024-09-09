# Sitemap Aggregation and Dashboard

This project aggregates sitemap URLs from various Indian news websites, extracts publication data, and visualizes it through a Looker dashboard. The process involves using Google Sheets to fetch and combine sitemap data, and then categorizing URLs by channel, format, and content type.

### Key Steps:
1. **Sitemap Extraction**: Using `IMPORTXML`, sitemap URLs and publication dates are fetched from 12 news websites.
2. **Data Processing**: Data from individual sheets is combined and categorized by channel, format (text, video, photo, web stories), and category (news, states, etc.).
3. **Visualization in Looker**: The processed data is connected to Looker to create dashboards that track URL publication trends and content distribution across various formats and channels.

---

