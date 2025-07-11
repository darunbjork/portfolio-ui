# Frontend-Backend Connectivity Analysis

## 🚨 **CRITICAL ISSUES IDENTIFIED**

### 1. **Network Error: `net::ERR_BLOCKED_BY_CLIENT`**
This error indicates one of the following:
- **Ad blocker** blocking API requests (most likely)
- **Security software** blocking connections
- **Backend server not running** on expected port
- **CORS configuration issues**

### 2. **API Configuration Analysis**

#### Frontend Configuration (`src/api/axios.ts`):
```javascript
const baseURL = import.meta.env.VITE_API_URL || 'http://localhost:3000/api/v1';
```

#### Expected Endpoints (from `src/api/services.ts`):
```javascript
// Authentication (✅ Working)
POST /auth/register
POST /auth/login
GET  /auth/me

// Projects (❌ Not working)
GET    /projects
POST   /projects
PUT    /projects/:id
DELETE /projects/:id

// Skills (❌ Not working)
GET    /skills
POST   /skills
PUT    /skills/:id
DELETE /skills/:id

// Experience (❌ Not working)
GET    /experience
POST   /experience
PUT    /experience/:id
DELETE /experience/:id
```

## 🔧 **IMMEDIATE SOLUTIONS**

### **Step 1: Check Backend Status**
```bash
# Check if your backend is running
curl http://localhost:3000/api/v1/projects

# Or check with the health endpoint
curl http://localhost:3000/health
```

### **Step 2: Disable Ad Blocker**
1. **uBlock Origin/AdBlock Plus**: Add `localhost` to whitelist
2. **Browser settings**: Disable shields/protection for localhost
3. **Antivirus software**: Whitelist localhost:3000

### **Step 3: Check CORS Configuration**
Your backend needs to allow requests from `http://localhost:5173` (your frontend port).

Expected backend CORS config:
```javascript
app.use(cors({
  origin: [
    'http://localhost:5173',
    'http://127.0.0.1:5173'
  ],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

### **Step 4: Verify Environment Variables**
Create `.env` file in your frontend root:
```bash
VITE_API_URL=http://localhost:3000/api/v1
```

## 🔍 **DIAGNOSTIC STEPS**

### **Test API Connectivity:**
```bash
# 1. Test basic connection
curl -v http://localhost:3000/api/v1/projects

# 2. Test with CORS headers
curl -H "Origin: http://localhost:5173" \
     -H "Access-Control-Request-Method: GET" \
     -H "Access-Control-Request-Headers: Content-Type" \
     -X OPTIONS \
     http://localhost:3000/api/v1/projects

# 3. Test authenticated endpoint (replace TOKEN)
curl -H "Authorization: Bearer YOUR_TOKEN" \
     http://localhost:3000/api/v1/projects
```

### **Browser Console Tests:**
Open browser console and run:
```javascript
// Test basic fetch
fetch('http://localhost:3000/api/v1/projects')
  .then(r => r.json())
  .then(console.log)
  .catch(console.error);

// Test with auth token (replace TOKEN)
fetch('http://localhost:3000/api/v1/projects', {
  headers: {
    'Authorization': 'Bearer YOUR_TOKEN',
    'Content-Type': 'application/json'
  }
})
.then(r => r.json())
.then(console.log)
.catch(console.error);
```

## 🔍 **BACKEND ANALYSIS NEEDED**

Since I couldn't access your backend repository directly, please verify your backend has:

### **Required Routes:**
```javascript
// Authentication routes (✅ Working)
app.post('/api/v1/auth/register', ...)
app.post('/api/v1/auth/login', ...)
app.get('/api/v1/auth/me', ...)

// Project routes (❌ Missing?)
app.get('/api/v1/projects', ...)
app.post('/api/v1/projects', ...)
app.put('/api/v1/projects/:id', ...)
app.delete('/api/v1/projects/:id', ...)

// Skills routes (❌ Missing?)
app.get('/api/v1/skills', ...)
app.post('/api/v1/skills', ...)
app.put('/api/v1/skills/:id', ...)
app.delete('/api/v1/skills/:id', ...)

// Experience routes (❌ Missing?)
app.get('/api/v1/experience', ...)
app.post('/api/v1/experience', ...)
app.put('/api/v1/experience/:id', ...)
app.delete('/api/v1/experience/:id', ...)
```

### **Required Middleware:**
```javascript
app.use(express.json());
app.use(cors({
  origin: ['http://localhost:5173', 'http://127.0.0.1:5173'],
  credentials: true
}));
```

### **Required Database Models:**
- `Project` model with fields: title, description, technologies[], githubUrl, liveUrl
- `Skill` model with fields: name, proficiency, category
- `ExperienceItem` model with fields: title, company, location, from, to, current, description

## 🎯 **QUICK FIX CHECKLIST**

- [ ] **Disable ad blocker** for localhost
- [ ] **Start backend server** on port 3000
- [ ] **Verify CORS** allows localhost:5173
- [ ] **Check routes exist** for /projects, /skills, /experience
- [ ] **Test auth token** is being sent correctly
- [ ] **Verify database connection** in backend
- [ ] **Check environment variables** in both frontend and backend

## 🚀 **Testing Commands**

After making changes, test each endpoint:

```bash
# 1. Test projects endpoint
curl http://localhost:3000/api/v1/projects

# 2. Test skills endpoint  
curl http://localhost:3000/api/v1/skills

# 3. Test experience endpoint
curl http://localhost:3000/api/v1/experience

# 4. Test with authentication
curl -H "Authorization: Bearer YOUR_TOKEN" \
     -H "Content-Type: application/json" \
     -X POST \
     -d '{"title":"Test","description":"Test desc","technologies":["React"]}' \
     http://localhost:3000/api/v1/projects
```

## 💡 **Why Only Auth Works**

The fact that login/register works but CRUD operations don't suggests:
1. **Auth endpoints exist** in your backend
2. **CRUD endpoints are missing** or have different paths
3. **Database/ORM not properly connected** for content management
4. **Middleware authentication** failing for protected routes

Start with disabling your ad blocker and verifying your backend is running with all required endpoints!