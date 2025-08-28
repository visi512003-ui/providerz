
# ProviderZ - Complete Project Structure & Code

## Project Overview
ProviderZ is a comprehensive platform connecting professionals with organizations across various industries including education, technology, healthcare, business, creative, engineering, and consulting sectors.

## Technology Stack
- **Backend**: Flask (Python)
- **Frontend**: HTML5, CSS3, JavaScript, Bootstrap 5
- **Database**: JSON file storage (data.json)
- **Features**: AI Chatbot, Professional profiles, Job postings, Booking system

## File Structure
```
ProviderZ/
├── main.py                    # Main Flask application
├── data.json                  # Data storage file
├── templates/
│   ├── base.html             # Base template with navigation
│   ├── index.html            # Homepage
│   ├── professionals.html    # Professional listings
│   ├── organizations.html    # Organization listings
│   ├── register_professional.html # Professional registration
│   ├── register_organization.html # Organization registration
│   ├── book_professional.html # Booking form
│   ├── bookings.html         # Bookings management
│   ├── job_posts.html        # Job listings
│   ├── post_job.html         # Job posting form
│   └── chatbot.html          # AI Assistant interface
└── static/                   # Static assets (handled via CDN)
```

## Main Application Code (main.py)

```python
from flask import Flask, render_template, request, redirect, url_for, flash, jsonify
import json
import os
from datetime import datetime

app = Flask(__name__)
app.secret_key = 'your-secret-key-here'

# Add custom JSON filter for Jinja2
@app.template_filter('tojsonfilter')
def to_json_filter(value):
    return json.dumps(value)

# Data storage (in production, use a proper database)
DATA_FILE = 'data.json'

# Job categories
JOB_CATEGORIES = {
    'education': ['Teacher', 'Professor', 'Tutor', 'Instructor', 'Curriculum Developer', 'Educational Consultant'],
    'technology': ['Software Developer', 'Data Scientist', 'Web Developer', 'IT Support', 'System Administrator', 'UI/UX Designer'],
    'healthcare': ['Nurse', 'Medical Assistant', 'Therapist', 'Healthcare Consultant', 'Medical Researcher', 'Pharmacist'],
    'business': ['Business Analyst', 'Project Manager', 'Marketing Specialist', 'HR Consultant', 'Financial Advisor', 'Operations Manager'],
    'creative': ['Graphic Designer', 'Content Writer', 'Photographer', 'Video Editor', 'Marketing Designer', 'Social Media Manager'],
    'engineering': ['Civil Engineer', 'Mechanical Engineer', 'Electrical Engineer', 'Software Engineer', 'Quality Assurance Engineer', 'Environmental Engineer'],
    'consulting': ['Management Consultant', 'Strategy Consultant', 'Training Consultant', 'Process Improvement Specialist', 'Change Management Specialist']
}

def load_data():
    if os.path.exists(DATA_FILE):
        with open(DATA_FILE, 'r') as f:
            return json.load(f)
    return {'professionals': [], 'organizations': [], 'bookings': [], 'job_posts': []}

def save_data(data):
    with open(DATA_FILE, 'w') as f:
        json.dump(data, f, indent=2)

@app.route('/')
def index():
    data = load_data()
    return render_template('index.html', 
                         professionals=data['professionals'], 
                         organizations=data['organizations'],
                         job_categories=JOB_CATEGORIES)

@app.route('/api/job-categories')
def api_job_categories():
    return jsonify(JOB_CATEGORIES)

@app.route('/professionals')
def professionals():
    data = load_data()
    category = request.args.get('category', 'all')
    search = request.args.get('search', '')
    
    filtered_professionals = data['professionals']
    
    if category != 'all':
        filtered_professionals = [p for p in filtered_professionals if p['category'] == category]
    
    if search:
        search_lower = search.lower()
        filtered_professionals = [p for p in filtered_professionals if 
                                search_lower in p['name'].lower() or 
                                search_lower in p['specialization'].lower() or
                                search_lower in p['skills'].lower()]
    
    return render_template('professionals.html', 
                         professionals=filtered_professionals,
                         job_categories=JOB_CATEGORIES,
                         current_category=category,
                         search_query=search)

@app.route('/organizations')
def organizations():
    data = load_data()
    return render_template('organizations.html', organizations=data['organizations'])

@app.route('/register_professional', methods=['GET', 'POST'])
def register_professional():
    if request.method == 'POST':
        data = load_data()
        professional = {
            'id': len(data['professionals']) + 1,
            'name': request.form['name'],
            'email': request.form['email'],
            'phone': request.form['phone'],
            'category': request.form['category'],
            'specialization': request.form['specialization'],
            'experience': request.form['experience'],
            'education': request.form['education'],
            'skills': request.form['skills'],
            'hourly_rate': request.form['hourly_rate'],
            'availability': request.form['availability'],
            'portfolio_link': request.form.get('portfolio_link', ''),
            'linkedin_profile': request.form.get('linkedin_profile', ''),
            'certifications': request.form.get('certifications', ''),
            'languages': request.form.get('languages', ''),
            'location': request.form['location'],
            'remote_work': request.form.get('remote_work') == 'on',
            'registration_date': datetime.now().isoformat()
        }
        data['professionals'].append(professional)
        save_data(data)
        flash('Professional registered successfully!', 'success')
        return redirect(url_for('professionals'))
    return render_template('register_professional.html', job_categories=JOB_CATEGORIES)

@app.route('/register_organization', methods=['GET', 'POST'])
def register_organization():
    if request.method == 'POST':
        data = load_data()
        organization = {
            'id': len(data['organizations']) + 1,
            'name': request.form['name'],
            'email': request.form['email'],
            'phone': request.form['phone'],
            'address': request.form['address'],
            'type': request.form['type'],
            'description': request.form['description']
        }
        data['organizations'].append(organization)
        save_data(data)
        flash('Organization registered successfully!', 'success')
        return redirect(url_for('organizations'))
    return render_template('register_organization.html')

@app.route('/book_professional/<int:professional_id>', methods=['GET', 'POST'])
def book_professional(professional_id):
    data = load_data()
    professional = next((p for p in data['professionals'] if p['id'] == professional_id), None)
    
    if not professional:
        flash('Professional not found!', 'error')
        return redirect(url_for('professionals'))
    
    if request.method == 'POST':
        booking = {
            'id': len(data['bookings']) + 1,
            'professional_id': professional_id,
            'organization_name': request.form['organization_name'],
            'contact_email': request.form['contact_email'],
            'contact_phone': request.form['contact_phone'],
            'start_date': request.form['start_date'],
            'end_date': request.form['end_date'],
            'project_type': request.form['project_type'],
            'requirements': request.form['requirements'],
            'budget': request.form['budget'],
            'booking_date': datetime.now().isoformat(),
            'status': 'pending'
        }
        data['bookings'].append(booking)
        save_data(data)
        flash('Booking request submitted successfully!', 'success')
        return redirect(url_for('professionals'))
    
    return render_template('book_professional.html', professional=professional)

@app.route('/bookings')
def bookings():
    data = load_data()
    # Enhance bookings with professional information
    enhanced_bookings = []
    for booking in data['bookings']:
        professional = next((p for p in data['professionals'] if p['id'] == booking['professional_id']), None)
        enhanced_booking = booking.copy()
        enhanced_booking['professional'] = professional
        enhanced_bookings.append(enhanced_booking)
    
    return render_template('bookings.html', bookings=enhanced_bookings)

@app.route('/job_posts')
def job_posts():
    data = load_data()
    return render_template('job_posts.html', job_posts=data['job_posts'])

@app.route('/post_job', methods=['GET', 'POST'])
def post_job():
    if request.method == 'POST':
        data = load_data()
        job_post = {
            'id': len(data['job_posts']) + 1,
            'title': request.form['title'],
            'company': request.form['company'],
            'category': request.form['category'],
            'description': request.form['description'],
            'requirements': request.form['requirements'],
            'salary_range': request.form['salary_range'],
            'location': request.form['location'],
            'remote_allowed': request.form.get('remote_allowed') == 'on',
            'employment_type': request.form['employment_type'],
            'contact_email': request.form['contact_email'],
            'deadline': request.form['deadline'],
            'posted_date': datetime.now().isoformat(),
            'status': 'active'
        }
        data['job_posts'].append(job_post)
        save_data(data)
        flash('Job posted successfully!', 'success')
        return redirect(url_for('job_posts'))
    
    return render_template('post_job.html', job_categories=JOB_CATEGORIES)

@app.route('/chatbot')
def chatbot():
    return render_template('chatbot.html')

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
```

## Design System & Styling

### Color Palette
- **Primary**: #F9ED69 (Yellow)
- **Secondary**: #F08A5D (Orange)
- **Accent**: #6A2C70 (Purple)
- **Success**: #B83B5E (Pink)
- **Dark**: #2d1b3d
- **Light**: #fdfbf7

### Typography
- **Primary Font**: Inter (Body text)
- **Display Font**: Poppins (Headings)

### Key Features

#### 1. Professional Registration System
- Multi-step form with validation
- Portfolio integration
- Skill categorization
- Rate and availability settings

#### 2. AI-Powered Chatbot
- Role-based interactions (Student, Teacher, Professional)
- Quick action buttons
- Contextual responses
- Typing indicators

#### 3. Advanced Search & Filtering
- Category-based filtering
- Text search across multiple fields
- Real-time results

#### 4. Booking Management
- Professional booking system
- Project type categorization
- Budget tracking
- Status management

#### 5. Job Board
- Multi-category job postings
- Remote work options
- Application deadline tracking

### UI Components

#### Interactive Elements
- 3D character models (👩‍🏫👨‍🎓👨‍💼)
- Floating animations
- Hover effects with transforms
- Gradient backgrounds

#### Navigation
- Responsive navbar
- Dropdown menus for categories
- Breadcrumb navigation
- Quick access buttons

### Data Structure

#### Professional Profile
```json
{
  "id": 1,
  "name": "John Doe",
  "email": "john@example.com",
  "category": "technology",
  "specialization": "Web Development",
  "experience": "5 years",
  "hourly_rate": "75",
  "skills": "JavaScript, React, Node.js",
  "remote_work": true
}
```

#### Booking System
```json
{
  "id": 1,
  "professional_id": 1,
  "organization_name": "Tech Corp",
  "project_type": "Long-term contract",
  "budget": "5000",
  "status": "pending"
}
```

### Deployment on Replit

1. **Setup**: All dependencies are managed through pyproject.toml
2. **Database**: JSON file storage for simplicity
3. **Port**: Configured for 0.0.0.0:5000
4. **Static Assets**: CDN-based (Bootstrap, FontAwesome)

### Future Enhancements

1. **Database Migration**: PostgreSQL integration
2. **Authentication**: User login/logout system
3. **Payment Integration**: Stripe/PayPal for transactions
4. **Real-time Chat**: WebSocket implementation
5. **File Uploads**: Portfolio and document management
6. **Email Notifications**: Booking confirmations
7. **Advanced Analytics**: Dashboard for insights

### Security Considerations

1. Form validation and sanitization
2. CSRF protection
3. Rate limiting for API endpoints
4. Input sanitization for XSS prevention
5. Secure file upload handling

### Performance Optimizations

1. Lazy loading for images
2. Minified CSS/JS
3. Gzip compression
4. Browser caching headers
5. Database indexing (future)

This project structure provides a solid foundation for a professional networking platform with room for scalability and additional features.
