# **Django Gas Utility Customer Service Backend**

## **📂 Django Project Structure**
A well-organized Django project follows the **Modular Monolith** pattern, separating concerns into different apps.

# Application Overview

The application will consist of two main interfaces:

1. **A customer-facing portal** where users can submit service requests, track their status, and manage their account information
2. **An administrative interface** for customer support representatives to manage and respond to service requests

## Core Features

### For Customers
- Account registration and management
- Service request submission with file attachments
- Request tracking and status updates
- Account information viewing and management

### For Customer Support Representatives
- Dashboard for viewing and managing service requests
- Tools for updating request status and communicating with customers
- Reporting and analytics capabilities

```
gas_utility/
│── manage.py
│── config/                      # Project Configuration
│   ├── __init__.py
│   ├── settings.py              # Django settings
│   ├── urls.py                  # Project-level URLs
│   ├── wsgi.py
│   ├── asgi.py
│
├── apps/
│   ├── accounts/                 # User management (Customer & Support Reps)
│   │   ├── models.py             # User model
│   │   ├── views.py              # Registration, Login, Profile
│   │   ├── urls.py
│   │   ├── forms.py
│   │   ├── serializers.py        # DRF serializers
│   │   ├── permissions.py        # API Permissions
│   │   ├── tests.py
│   │   ├── admin.py
│   │   ├── signals.py            # Custom user signals
│   │   ├── templates/accounts/   # HTML templates for frontend
│   │   └── static/accounts/      # CSS, JS, images
│
│   ├── service_requests/         # Service Requests
│   │   ├── models.py             # ServiceRequest, Attachment models
│   │   ├── views.py              # Views for requests
│   │   ├── urls.py               # Service request endpoints
│   │   ├── serializers.py        # API Serializers
│   │   ├── forms.py              # Forms for request submission
│   │   ├── permissions.py        # Custom permissions
│   │   ├── tests.py              # Unit tests
│   │   ├── admin.py              # Admin configurations
│   │   ├── templates/service_requests/
│   │   ├── static/service_requests/
│   │   └── signals.py            # Signals for notifications
│
│   ├── dashboard/                # Admin & Support Dashboard
│   │   ├── views.py              # Dashboards for Admins & Support Agents
│   │   ├── urls.py
│   │   ├── templates/dashboard/
│   │   ├── static/dashboard/
│   │   └── permissions.py        # Access Control
│
│   ├── notifications/            # Email & Push Notifications
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── signals.py
│   │   ├── tasks.py              # Celery Tasks for Async notifications
│   │   ├── templates/notifications/
│   │   ├── urls.py
│   │   └── serializers.py
│
├── templates/                    # Global templates
├── static/                        # Global static files
├── media/                         # User-uploaded files
├── requirements.txt               # Dependencies
└── README.md                      # Project documentation
```

---

## **🚀 Key Features**

### **1️⃣ Service Requests**
- Customers can submit a request with:
  - **Type of request** (Gas leak, Installation, Billing issue, etc.)
  - **Details about the request**
  - **File attachments** (Images, PDFs)
- Request status updates: `Pending → In Progress → Resolved`

#### **Model: `ServiceRequest`**
```python
from django.db import models
from django.contrib.auth.models import User

class ServiceRequest(models.Model):
    STATUS_CHOICES = [
        ('pending', 'Pending'),
        ('in_progress', 'In Progress'),
        ('resolved', 'Resolved')
    ]

    user = models.ForeignKey(User, on_delete=models.CASCADE)
    title = models.CharField(max_length=255)
    description = models.TextField()
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default='pending')
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return f"{self.title} - {self.status}"
```

#### **Serializer for API:**
```python
from rest_framework import serializers
from .models import ServiceRequest

class ServiceRequestSerializer(serializers.ModelSerializer):
    class Meta:
        model = ServiceRequest
        fields = '__all__'
```

---

### **2️⃣ Request Tracking**
- Customers can **view the status** of their requests.
- Email/Push notifications for status updates.

#### **View for Request Tracking:**
```python
from rest_framework import viewsets
from rest_framework.permissions import IsAuthenticated
from .models import ServiceRequest
from .serializers import ServiceRequestSerializer

class ServiceRequestViewSet(viewsets.ModelViewSet):
    queryset = ServiceRequest.objects.all()
    serializer_class = ServiceRequestSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return self.queryset.filter(user=self.request.user)
```

#### **URL Patterns**
```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import ServiceRequestViewSet

router = DefaultRouter()
router.register(r'service-requests', ServiceRequestViewSet)

urlpatterns = [
    path('', include(router.urls)),
]
```

---

### **3️⃣ Customer Dashboard**
- Customers can log in to **view submitted requests**.
- Admin and support representatives can **manage and resolve requests**.

#### **Admin Dashboard View**
```python
from django.contrib.auth.decorators import login_required
from django.shortcuts import render
from service_requests.models import ServiceRequest

@login_required
def admin_dashboard(request):
    requests = ServiceRequest.objects.all()
    return render(request, 'dashboard/admin_dashboard.html', {'requests': requests})
```

---

### **4️⃣ Notifications (Email & Push)**
- Customers receive **email notifications** when their request status changes.
- Admins get alerts for **new requests**.

#### **Django Signal for Email Notifications**
```python
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.core.mail import send_mail
from .models import ServiceRequest

@receiver(post_save, sender=ServiceRequest)
def send_status_update_email(sender, instance, **kwargs):
    subject = f"Service Request Update - {instance.title}"
    message = f"Your request status has been updated to {instance.status}."
    recipient_email = instance.user.email
    send_mail(subject, message, 'support@gasutility.com', [recipient_email])
```
---

## **🛠️ Next Steps**
1. Implement **React or Vue.js frontend** for better UX.
2. Add **Twilio SMS or Firebase push notifications**.
3. Use **GraphQL** for more flexible API queries.

