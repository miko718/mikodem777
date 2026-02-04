# מדריך אינטגרציית Google Maps API - Mikodem

## סקירה כללית
מסמך זה מפרט את האינטגרציה של Google Maps Platform באפליקציית Mikodem לניהול תורים חכם.

---

## 🔑 APIs נדרשים מ-Google Cloud Platform

### 1. **Google Maps JavaScript API** (קריטי)
**תפקיד:** הצגת מפה אינטראקטיבית בממשק המשתמש

**שימוש באפליקציה:**
- הצגת המפה במסך הלקוח
- הצגת סמני מיקום (העסק והלקוח)
- עיצוב מותאם אישית של המפה

**הערות טכניות:**
- נטען באמצעות: `<script src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&libraries=places,geometry&language=he">`
- כולל ספריות: `places` (לחיפוש כתובות), `geometry` (לחישובי מרחק)

---

### 2. **Google Directions API** (קריטי)
**תפקיד:** חישוב מסלול ניווט בין שתי נקודות

**שימוש באפליקציה:**
- הצגת המסלול על המפה מהלקוח לעסק
- חישוב זמן הגעה משוער (ETA)
- התחשבות בתנועה בזמן אמת

**דוגמת קוד:**
```javascript
const directionsService = new google.maps.DirectionsService();
const request = {
    origin: userLocation,
    destination: businessLocation,
    travelMode: google.maps.TravelMode.DRIVING,
    drivingOptions: {
        departureTime: new Date(),
        trafficModel: 'bestguess'
    }
};

directionsService.route(request, (result, status) => {
    if (status === 'OK') {
        // עיבוד התוצאה
        const route = result.routes[0].legs[0];
        console.log('מרחק:', route.distance.text);
        console.log('זמן:', route.duration_in_traffic.text);
    }
});
```

---

### 3. **Google Distance Matrix API** (קריטי - המוח של המערכת)
**תפקיד:** חישוב מרחקים וזמני נסיעה בין מספר מקורות ליעדים

**שימוש באפליקציה:**
- **הלוגיקה המרכזית של Swap:** השרת דוגם מיקומי לקוחות כל כמה דקות
- מחשב ETA מדויק עם תנועה בזמן אמת
- מזהה לקוחות שיאחרו לתור
- מוצא לקוחים אחרים שקרובים יותר להחלפה

**דוגמת Request (צד שרת - Node.js):**
```javascript
const axios = require('axios');

async function calculateDistance(origins, destinations) {
    const url = 'https://maps.googleapis.com/maps/api/distancematrix/json';
    const params = {
        origins: origins.join('|'),
        destinations: destinations.join('|'),
        mode: 'driving',
        departure_time: 'now',
        traffic_model: 'best_guess',
        key: process.env.GOOGLE_MAPS_API_KEY
    };
    
    const response = await axios.get(url, { params });
    return response.data;
}

// דוגמת שימוש
const origins = ['32.0853,34.8353']; // מיקום הלקוח
const destinations = ['32.0853,34.7818']; // מיקום העסק

const result = await calculateDistance(origins, destinations);
const duration = result.rows[0].elements[0].duration_in_traffic.value; // בשניות
```

**חשוב:** API זה יקר יחסית - יש לממש Caching חכם!

---

### 4. **Google Places API** (חשוב)
**תפקיד:** חיפוש מקומות והשלמה אוטומטית של כתובות

**שימוש באפליקציה:**
- **הרשמת בעלי עסקים:** השלמת כתובת העסק אוטומטית
- **הגדרות לקוח:** שמירת כתובות בית/עבודה
- אימות כתובות

**דוגמת קוד - Autocomplete:**
```javascript
const input = document.getElementById('address-input');
const autocomplete = new google.maps.places.Autocomplete(input, {
    componentRestrictions: { country: 'il' }, // הגבלה לישראל
    fields: ['address_components', 'geometry', 'name']
});

autocomplete.addListener('place_changed', () => {
    const place = autocomplete.getPlace();
    if (place.geometry) {
        const lat = place.geometry.location.lat();
        const lng = place.geometry.location.lng();
        console.log('כתובת נבחרה:', place.name, lat, lng);
    }
});
```

---

### 5. **Google Geocoding API** (חשוב)
**תפקיד:** המרה בין כתובות טקסטואליות לקואורדינטות (ולהיפך)

**שימוש באפליקציה:**
- המרת כתובת העסק ל-Lat/Long לשמירה במסד נתונים
- המרת קואורדינטות GPS להצגת כתובת קריאה

**דוגמת קוד:**
```javascript
// המרת כתובת לקואורדינטות
const geocoder = new google.maps.Geocoder();
const address = 'הרצל 45, תל אביב';

geocoder.geocode({ address: address }, (results, status) => {
    if (status === 'OK') {
        const location = results[0].geometry.location;
        console.log('Lat:', location.lat(), 'Lng:', location.lng());
    }
});

// המרת קואורדינטות לכתובת
const latlng = { lat: 32.0853, lng: 34.7818 };
geocoder.geocode({ location: latlng }, (results, status) => {
    if (status === 'OK') {
        console.log('כתובת:', results[0].formatted_address);
    }
});
```

---

## 📋 הוראות התקנה ב-Google Cloud Console

### שלב 1: יצירת פרויקט
1. היכנס ל-[Google Cloud Console](https://console.cloud.google.com)
2. לחץ על "Select a project" → "New Project"
3. תן שם לפרויקט: `Mikodem-Production`
4. לחץ "Create"

### שלב 2: הפעלת APIs
1. בתפריט צד שמאל: **APIs & Services** → **Library**
2. חפש והפעל את הבאים (לחץ "ENABLE" בכל אחד):
   - ✅ Maps JavaScript API
   - ✅ Directions API
   - ✅ Distance Matrix API
   - ✅ Places API
   - ✅ Geocoding API

### שלב 3: יצירת API Key
1. **APIs & Services** → **Credentials**
2. לחץ **Create Credentials** → **API Key**
3. **חשוב מאוד:** לחץ "Restrict Key" מיד לאחר היצירה

### שלב 4: הגבלת המפתח (אבטחה קריטית!)

#### לאפליקציית Web:
```
Application restrictions:
- HTTP referrers (web sites)
- Add referrer: https://mikodem.app/*
- Add referrer: http://localhost:3000/* (לפיתוח בלבד)
```

#### לאפליקציית מובייל (iOS/Android):
```
Application restrictions:
- Android apps / iOS apps
- Add package name: com.mikodem.app
- Add SHA-1 fingerprint: [הציג מפתח דיבוג שלך]
```

#### לשרת Backend:
```
Application restrictions:
- IP addresses
- Add server IP: 203.0.113.1 (דוגמה)
```

#### הגבלת APIs:
```
API restrictions:
- Restrict key
- Select: Maps JavaScript API, Directions API, Distance Matrix API, Places API, Geocoding API
```

---

## 🔐 אבטחת מפתחות API

### ⚠️ אל תעשה NEVER:
```javascript
// ❌ לעולם לא להכליל מפתח בקוד!
const API_KEY = 'AIzaSyC-xxxxxxxxxxxxxxxxxxxxx';
```

### ✅ דרך נכונה - Environment Variables:

#### Frontend (.env):
```bash
VITE_GOOGLE_MAPS_API_KEY=AIzaSyC-xxxxxxxxxxxxxxxxxxxxx
```

#### Backend (.env):
```bash
GOOGLE_MAPS_API_KEY=AIzaSyC-xxxxxxxxxxxxxxxxxxxxx
```

#### שימוש ב-React:
```javascript
const apiKey = import.meta.env.VITE_GOOGLE_MAPS_API_KEY;
```

#### שימוש ב-Node.js:
```javascript
require('dotenv').config();
const apiKey = process.env.GOOGLE_MAPS_API_KEY;
```

---

## 💰 אופטימיזציה וחיסכון בעלויות

### מחירון Google Maps (נכון ל-2024):
- **Maps JavaScript API:** $7 לכל 1,000 טעינות מפה
- **Directions API:** $5 לכל 1,000 בקשות
- **Distance Matrix API:** $5 לכל 1,000 אלמנטים
- **Places API - Autocomplete:** $2.83 לכל 1,000 בקשות
- **Geocoding API:** $5 לכל 1,000 בקשות

**קרדיט חינמי:** Google נותנת $200/חודש חינם!

### אסטרטגיות לחיסכון:

#### 1. Caching של תוצאות Distance Matrix
```javascript
const cache = new Map();

async function getCachedDistance(origin, destination) {
    const key = `${origin}-${destination}`;
    
    // בדיקת cache
    if (cache.has(key)) {
        const cached = cache.get(key);
        // תוקף 5 דקות
        if (Date.now() - cached.timestamp < 5 * 60 * 1000) {
            return cached.data;
        }
    }
    
    // קריאה ל-API
    const result = await fetchDistanceMatrix(origin, destination);
    cache.set(key, { data: result, timestamp: Date.now() });
    return result;
}
```

#### 2. דגימת מיקום חכמה
```javascript
// ❌ לא יעיל - בודק כל לקוח כל דקה
setInterval(() => {
    allClients.forEach(client => checkLocation(client));
}, 60000);

// ✅ יעיל - בודק רק לקוחים רלוונטיים
setInterval(() => {
    const relevantClients = allClients.filter(client => {
        const timeToAppointment = client.appointmentTime - Date.now();
        return timeToAppointment > 0 && timeToAppointment < 60 * 60 * 1000; // שעה לפני
    });
    
    relevantClients.forEach(client => checkLocation(client));
}, 5 * 60 * 1000); // כל 5 דקות
```

#### 3. Batch Requests
```javascript
// ❌ 10 קריאות API נפרדות
for (let client of clients) {
    await getDistance(client.location, businessLocation);
}

// ✅ קריאה אחת עם 10 origins
const origins = clients.map(c => c.location);
const result = await getDistanceMatrix(origins, [businessLocation]);
```

#### 4. הגבלת תדירות דגימה
```javascript
function shouldCheckLocation(client) {
    const timeToAppointment = client.appointmentTime - Date.now();
    
    // 1 שעה לפני - בדוק כל 10 דקות
    if (timeToAppointment < 60 * 60 * 1000) return true;
    
    // 2 שעות לפני - בדוק כל 30 דקות
    if (timeToAppointment < 2 * 60 * 60 * 1000) return true;
    
    // רחוק מדי - אל תבדוק
    return false;
}
```

---

## 🏗️ ארכיטקטורה מומלצת

### Client-Side (React/Vue):
```javascript
// טעינת מפה והצגה בלבד
- Google Maps JavaScript API
- הצגת מסלול (Directions API)
- Autocomplete לכתובות (Places API)
```

### Server-Side (Node.js/Python):
```javascript
// הלוגיקה העסקית והחישובים
- Distance Matrix API (דגימה כל 5 דקות)
- Geocoding API (המרת כתובות)
- שמירת תוצאות ב-Redis/Database
```

### מסד נתונים:
```sql
CREATE TABLE locations (
    id INT PRIMARY KEY,
    business_id INT,
    address TEXT,
    latitude DECIMAL(10, 8),
    longitude DECIMAL(11, 8),
    created_at TIMESTAMP
);

CREATE TABLE distance_cache (
    origin_lat DECIMAL(10, 8),
    origin_lng DECIMAL(11, 8),
    dest_lat DECIMAL(10, 8),
    dest_lng DECIMAL(11, 8),
    distance_km DECIMAL(5, 2),
    duration_minutes INT,
    duration_in_traffic_minutes INT,
    cached_at TIMESTAMP,
    PRIMARY KEY (origin_lat, origin_lng, dest_lat, dest_lng)
);
```

---

## 🧪 בדיקת התקנה

### טסט 1: טעינת מפה בסיסית
```html
<!DOCTYPE html>
<html>
<head>
    <title>Test Maps</title>
    <script src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY"></script>
</head>
<body>
    <div id="map" style="width:100%;height:400px"></div>
    <script>
        const map = new google.maps.Map(document.getElementById('map'), {
            center: { lat: 32.0853, lng: 34.7818 },
            zoom: 14
        });
        console.log('✅ Map loaded successfully!');
    </script>
</body>
</html>
```

### טסט 2: Distance Matrix (Node.js)
```javascript
const axios = require('axios');

async function testDistanceMatrix() {
    const url = 'https://maps.googleapis.com/maps/api/distancematrix/json';
    try {
        const response = await axios.get(url, {
            params: {
                origins: '32.0853,34.8353',
                destinations: '32.0853,34.7818',
                key: process.env.GOOGLE_MAPS_API_KEY
            }
        });
        console.log('✅ Distance Matrix working!');
        console.log(response.data);
    } catch (error) {
        console.error('❌ Error:', error.message);
    }
}

testDistanceMatrix();
```

---

## 📊 מעקב ושימוש

### Google Cloud Console - Monitoring:
1. **APIs & Services** → **Dashboard**
2. צפה בגרפים:
   - Requests per day
   - Errors
   - Latency
   
### הגדרת התראות:
```
Budget & Alerts:
- Set budget: $50/month
- Alert at: 50%, 90%, 100%
- Email: dev@mikodem.app
```

---

## ⚡ טיפים נוספים

1. **שמירת סטטיסטיקות:** שמור מידע על דפוסי שימוש כדי לזהות ימים/שעות עמוסים
2. **Fallback למיקום:** אם GPS לא זמין, השתמש ב-IP geolocation כחלופה
3. **Offline Support:** שמור מפות במטמון למצבים ללא אינטרנט
4. **בדיקת דיוק:** בדוק את `accuracy` של מיקום GPS - דלג על מיקומים לא מדויקים

---

## 📞 תמיכה ומשאבים

- [Google Maps Platform Documentation](https://developers.google.com/maps/documentation)
- [Distance Matrix API Guide](https://developers.google.com/maps/documentation/distance-matrix)
- [Google Maps Platform Support](https://developers.google.com/maps/support)
- [Stack Overflow - google-maps tag](https://stackoverflow.com/questions/tagged/google-maps)

---

## ✅ רשימת משימות למפתח

- [ ] יצירת פרויקט ב-Google Cloud Console
- [ ] הפעלת 5 APIs הנדרשים
- [ ] יצירת API Keys (נפרד לכל סביבה: dev/staging/prod)
- [ ] הגדרת הגבלות אבטחה למפתחות
- [ ] מימוש Caching ל-Distance Matrix
- [ ] הגדרת Environment Variables
- [ ] בניית endpoint בשרת לדגימת מיקומים
- [ ] יצירת מסד נתונים עם טבלאות locations ו-distance_cache
- [ ] הגדרת cron job לניקוי cache ישן
- [ ] בדיקות E2E לכל תרחיש
- [ ] הגדרת מעקב ותקציב ב-Google Cloud
- [ ] תיעוד API למפתחים עתידיים

---

**גרסה:** 1.0  
**תאריך עדכון אחרון:** פברואר 2026  
**מחבר:** צוות Mikodem
