# Mikodem 777

ניהול תורים חכם בזמן אמת – מסך לקוח ומסך בעל עסק.

## הרצה

פתח את `index.html` בדפדפן (או הפעל שרת מקומי):

```bash
# עם Python
python3 -m http.server 8765
# ואז גלוש ל־http://localhost:8765
```

## תוכן

- **מסך לקוח**: מצב איחור (פקק), הצעת החלפת תור, מסך הצלחה / "אסתדר".
- **מסך בעל עסק**: רשימת תורים להיום, סטטוס ומרחק, סטטיסטיקות.

React 18 + Babel (standalone), קובץ HTML יחיד.

## גרסה עם Google Maps

- **`mikodem-app-with-maps.html`** – גרסה עם מפת Google אמיתית, Directions API (מסלול + זמן נסיעה), והזנת מפתח API. מתאים לפיתוח ואינטגרציה.
- **`google-maps-integration-guide.md`** – מדריך מלא לאינטגרציית Google Maps (Maps JavaScript, Directions, Distance Matrix, Places, Geocoding), התקנה ב-Google Cloud, אבטחה ואופטימיזציה.
