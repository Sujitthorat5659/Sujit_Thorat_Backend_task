# Sujit_Thorat_Backend_task

Application Structure

1. Models (models.py)

from django.db import models
from django.contrib.auth.models import User

class ServiceRequest(models.Model):
    SERVICE_TYPES = [
        ('repair', 'Repair'),
        ('installation', 'Installation'),
        ('billing', 'Billing Inquiry'),
    ]
    
    customer = models.ForeignKey(User, on_delete=models.CASCADE)
    service_type = models.CharField(max_length=20, choices=SERVICE_TYPES)
    description = models.TextField()
    file_attachment = models.FileField(upload_to='attachments/', blank=True, null=True)
    status = models.CharField(max_length=20, default='submitted')
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

class RequestStatus(models.Model):
    service_request = models.ForeignKey(ServiceRequest, on_delete=models.CASCADE)
    status = models.CharField(max_length=20)
    timestamp = models.DateTimeField(auto_now_add=True)
    csr_notes = models.TextField(blank=True, null=True)

2. Views (views.py)
Defines the logic behind handling user requests and displaying the pages.

from django.shortcuts import render, redirect
from .models import ServiceRequest, RequestStatus
from .forms import ServiceRequestForm, RequestUpdateForm

def submit_request(request):
    if request.method == 'POST':
        form = ServiceRequestForm(request.POST, request.FILES)
        if form.is_valid():
            service_request = form.save(commit=False)
            service_request.customer = request.user
            service_request.save()
            return redirect('track_request', id=service_request.id)
    else:
        form = ServiceRequestForm()
    return render(request, 'submit_request.html', {'form': form})

def track_request(request, id):
    service_request = ServiceRequest.objects.get(id=id)
    request_statuses = RequestStatus.objects.filter(service_request=service_request)
    return render(request, 'track_request.html', {
        'service_request': service_request,
        'request_statuses': request_statuses
    })

def manage_requests(request):
    if request.user.is_staff:  # Check if user is a CSR
        all_requests = ServiceRequest.objects.all()
        return render(request, 'manage_requests.html', {'all_requests': all_requests})
    return redirect('home')

3. Forms (forms.py)
Handles the customer input for submitting requests and CSR updates.

from django import forms
from .models import ServiceRequest, RequestStatus

class ServiceRequestForm(forms.ModelForm):
    class Meta:
        model = ServiceRequest
        fields = ['service_type', 'description', 'file_attachment']

class RequestUpdateForm(forms.ModelForm):
    class Meta:
        model = RequestStatus
        fields = ['status', 'csr_notes']
4. Templates (templates/)
submit_request.html: Form for customers to submit service requests.
track_request.html: Displays the request’s status and any updates for the customer.
manage_requests.html: Dashboard for CSRs to manage and update service requests.

5. URLs (urls.py)
Defines the routing paths for the application.

from django.urls import path
from . import views

urlpatterns = [
    path('submit_request/', views.submit_request, name='submit_request'),
    path('track_request/<int:id>/', views.track_request, name='track_request'),
    path('admin/manage_requests/', views.manage_requests, name='manage_requests'),
]
