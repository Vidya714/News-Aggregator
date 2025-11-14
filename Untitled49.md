```python
import tkinter as tk
from tkinter import ttk, messagebox
from PIL import Image, ImageTk
import os  # To check if file exists
import feedparser
import webbrowser
import re  # For removing HTML tags

# Define news sources with their respective categories and RSS feed URLs
NEWS_SOURCES = {
    "The Times of India": {
        "Sports": "https://timesofindia.indiatimes.com/rssfeeds/4719148.cms",
        "Politics": "https://timesofindia.indiatimes.com/rssfeeds/296589292.cms",
        "Technology": "https://timesofindia.indiatimes.com/rssfeeds/5880659.cms",
        "India": "https://timesofindia.indiatimes.com/rssfeeds/-2128936835.cms",
        "World": "https://timesofindia.indiatimes.com/rssfeeds/296589292.cms",
    },
    "Sakshi": {
        "Sports": "https://www.sakshi.com/rss/sports.xml",
        "Politics": "https://www.sakshi.com/rss/politics.xml",
        "Entertainment": "https://www.sakshi.com/rss/entertainment.xml",
    },
}

BACKGROUND_IMAGE_PATH = ("C:\\Users\\dheek\\Downloads\\news-studio-background.jpg")  # Ensure you have a valid image path

# Function to fetch news articles
def fetch_articles(feed_url):
    feed = feedparser.parse(feed_url)
    articles = []
    for entry in feed.entries[:10]:  # Fetch top 10 articles
        raw_summary = entry.get("summary", "No summary available")
        clean_summary = re.sub(r'<.*?>', '', raw_summary).replace("\n", " ").strip()  # Remove HTML tags

        articles.append({
            "Title": entry.title,
            "URL": entry.link,
            "Published": entry.get("published", "Unknown Date"),
            "Summary": clean_summary
        })
    return articles

# Function to fetch news and display it in a new window
def fetch_news():
    source = source_combo.get()
    category = column_combo.get()

    if not source or not category:
        messagebox.showerror("Error", "Please select both a news source and a category.")
        return

    feed_url = NEWS_SOURCES[source].get(category)
    if not feed_url:
        messagebox.showerror("Error", "Invalid selection.")
        return

    articles = fetch_articles(feed_url)

    if not articles:
        messagebox.showinfo("No Articles", "No articles found.")
        return

    # Create a new window to display the news
    news_window = tk.Toplevel(root)
    news_window.title(f"News from {source} - {category}")
    news_window.geometry("800x600")

    canvas = tk.Canvas(news_window)
    scrollbar = ttk.Scrollbar(news_window, orient="vertical", command=canvas.yview)
    scroll_frame = tk.Frame(canvas)

    scroll_frame.bind("<Configure>", lambda e: canvas.configure(scrollregion=canvas.bbox("all")))
    canvas.create_window((0, 0), window=scroll_frame, anchor="nw")
    canvas.configure(yscrollcommand=scrollbar.set)

    # ✅ Enable mouse wheel scrolling
    def _on_mousewheel(event):
        canvas.yview_scroll(int(-1 * (event.delta / 120)), "units")

    canvas.bind_all("<MouseWheel>", _on_mousewheel)      # Windows
    canvas.bind_all("<Button-4>", lambda e: canvas.yview_scroll(-1, "units"))  # Linux scroll up
    canvas.bind_all("<Button-5>", lambda e: canvas.yview_scroll(1, "units"))   # Linux scroll down


    for article in articles:
        title_label = tk.Label(scroll_frame, text=article["Title"], font=("Arial", 12, "bold"), fg="black", cursor="hand2")
        title_label.pack(anchor="w", padx=10, pady=5)
        title_label.bind("<Button-1>", lambda e, url=article["URL"]: webbrowser.open(url))

        url_label = tk.Label(scroll_frame, text=f"URL: {article['URL']}", font=("Arial", 10), fg="blue", cursor="hand2")
        url_label.pack(anchor="w", padx=10, pady=2)
        url_label.bind("<Button-1>", lambda e, url=article["URL"]: webbrowser.open(url))

        published_label = tk.Label(scroll_frame, text=f"Published: {article['Published']}", font=("Arial", 10), fg="black")
        published_label.pack(anchor="w", padx=10, pady=2)

        summary_label = tk.Label(scroll_frame, text=f"Summary: {article['Summary']}", font=("Arial", 10), wraplength=750, justify="left",fg="black")
        summary_label.pack(anchor="w", padx=10, pady=5)

        separator = tk.Frame(scroll_frame, height=2, bg="black")
        separator.pack(fill="x", padx=5, pady=5)

    canvas.pack(side="left", fill="both", expand=True)
    scrollbar.pack(side="right", fill="y")

# Function to set background image
def set_background():
    global bg_label, bg_photo
    if not os.path.exists(BACKGROUND_IMAGE_PATH):
        messagebox.showerror("Error", f"Background image not found: {BACKGROUND_IMAGE_PATH}")
        return
    bg_image = Image.open(BACKGROUND_IMAGE_PATH)
    bg_image = bg_image.resize((1600, 1400), Image.LANCZOS)
    bg_photo = ImageTk.PhotoImage(bg_image)
    bg_label.config(image=bg_photo)
    bg_label.image = bg_photo  # Keep reference to avoid garbage collection

# GUI Setup
def create_gui():
    global root, bg_label, bg_photo, source_combo, column_combo
    root = tk.Tk()
    root.title("News Aggregator")
    root.geometry("1200x900")

    # Create a label for the background before setting it
    bg_label = tk.Label(root)
    bg_label.place(x=0, y=0, relwidth=1, relheight=1)
    
    # Now set the background image
    root.after(100, set_background)  # Delay setting background to ensure mainloop is running
    
    # Main frame
    main_frame = tk.Frame(root, bg="white", bd=2, relief=tk.RIDGE, width=600, height=400)
    main_frame.place(relx=0.5, rely=0.5, anchor=tk.CENTER, relwidth=0.5, relheight=0.5)

    # Title
    title_label = tk.Label(main_frame, text="Personalized News Aggregator", font=("Bold", 20, "bold"), bg="blue", fg="white", padx=20, pady=10)
    title_label.pack(pady=10, fill=tk.X)

    # Select the News Source
    source_label = tk.Label(main_frame, text="Select a News Source:", font=("Arial", 12))
    source_label.pack(pady=10)
    source_combo = ttk.Combobox(main_frame, values=list(NEWS_SOURCES.keys()), state="readonly", font=("Arial", 12))
    source_combo.pack(pady=10)

    # Select Article
    column_label = tk.Label(main_frame, text="Select a Category:", font=("Arial", 12))
    column_label.pack(pady=5)
    column_combo = ttk.Combobox(main_frame, state="readonly", font=("Arial", 12))
    column_combo.pack(pady=5)
    source_combo.bind("<<ComboboxSelected>>", lambda event: column_combo.config(values=list(NEWS_SOURCES[source_combo.get()].keys())))

    # Fetch the Article
    fetch_button = tk.Button(main_frame, text="Fetch News", font=("Arial", 12), bg="green", fg="white", command=fetch_news)
    fetch_button.pack(pady=10)

    root.mainloop()

if __name__ == "__main__":
    create_gui()
```


```python

```
