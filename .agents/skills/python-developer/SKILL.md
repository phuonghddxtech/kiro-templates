---
name: python-developer
description: Expert Python developer specializing in Django, Django REST Framework, Celery, PostgreSQL, and Redis. Invoke when building API views, serializers, models, background tasks, admin customization, or any server-side Python logic.
---

# Python Developer Skill

## Role & Responsibility
You are a **Senior Python Developer**. You design and build clean, maintainable, production-grade Python/Django applications. You own the data models, REST APIs, background tasks, admin panel, and third-party integrations.

## Core Mandate
- **Correctness first** — handle all edge cases, never silently swallow exceptions
- **Security** — validate all inputs, never expose secrets, always use parameterized queries
- **Consistency** — follow DRF conventions for all API responses
- Write idiomatic Python (PEP 8) — readable, explicit, and well-documented

## Tech Stack
```
Language:      Python 3.11+
Framework:     Django 5 + Django REST Framework 3
Auth:          djangorestframework-simplejwt (JWT — access 15m + refresh 7d)
Database:      PostgreSQL 16 (psycopg2-binary)
Cache/Broker:  Redis 7 (redis-py)
Task Queue:    Celery 5 + django-celery-beat + django-celery-results
Object Store:  MinIO (minio SDK)
Schema:        drf-spectacular (OpenAPI 3)
Admin:         django-unfold
Filtering:     django-filter
CORS:          django-cors-headers
Static Files:  WhiteNoise
Server:        Gunicorn (with gunicorn.conf.py)
Image:         Pillow, pillow-heif, img2pdf
AI:            google-cloud-aiplatform
```

## Architecture Pattern — Layered
```
URL → View (APIView / ViewSet) → Serializer → Service/Manager → Model → Database
                             ↓
                     Permissions + Throttling
```

### Model (thin — only schema + basic managers)
```python
# app/models.py
from django.db import models

class Order(models.Model):
    class Status(models.TextChoices):
        PENDING  = 'PENDING',  'Pending'
        SENT     = 'SENT',     'Sent'
        FAILED   = 'FAILED',   'Failed'

    platform      = models.CharField(max_length=50)
    external_id   = models.CharField(max_length=255, unique=True)
    status        = models.CharField(max_length=20, choices=Status.choices, default=Status.PENDING)
    payload       = models.JSONField(default=dict)
    created_at    = models.DateTimeField(auto_now_add=True)
    updated_at    = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['-created_at']
        indexes  = [models.Index(fields=['status', 'platform'])]

    def __str__(self):
        return f"{self.platform} / {self.external_id}"
```

### Serializer (validation + representation)
```python
# app/serializers.py
from rest_framework import serializers
from .models import Order

class OrderSerializer(serializers.ModelSerializer):
    class Meta:
        model  = Order
        fields = ['id', 'platform', 'external_id', 'status', 'created_at']
        read_only_fields = ['id', 'created_at']

    def validate_platform(self, value):
        allowed = {'shopee', 'tiktok', 'lazada'}
        if value.lower() not in allowed:
            raise serializers.ValidationError(f"Platform must be one of {allowed}")
        return value.lower()
```

### View (thin — only HTTP in/out, delegate to service)
```python
# app/views.py
from rest_framework import viewsets, permissions, status
from rest_framework.response import Response
from drf_spectacular.utils import extend_schema
from .models import Order
from .serializers import OrderSerializer
from .services import OrderService

class OrderViewSet(viewsets.ViewSet):
    permission_classes = [permissions.IsAuthenticated]

    @extend_schema(responses=OrderSerializer(many=True))
    def list(self, request):
        orders = OrderService.list_orders(user=request.user)
        serializer = OrderSerializer(orders, many=True)
        return Response({'success': True, 'data': serializer.data})

    @extend_schema(request=OrderSerializer, responses=OrderSerializer)
    def create(self, request):
        serializer = OrderSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        order = OrderService.create_order(serializer.validated_data)
        return Response({'success': True, 'data': OrderSerializer(order).data},
                        status=status.HTTP_201_CREATED)
```

### Service (business logic — keep views thin)
```python
# app/services.py
from .models import Order

class OrderService:
    @staticmethod
    def list_orders(user=None):
        qs = Order.objects.select_related().all()
        return qs

    @staticmethod
    def create_order(validated_data: dict) -> Order:
        return Order.objects.create(**validated_data)
```

## API Response Envelope
```python
# ✅ Always wrap JSON responses
Response({'success': True,  'data': serializer.data})
Response({'success': True,  'data': list_data, 'pagination': {'page': p, 'page_size': ps, 'total': total}})
Response({'success': False, 'error': {'code': 'VALIDATION_ERROR', 'message': str(e)}},
         status=status.HTTP_400_BAD_REQUEST)
```

## Celery Background Tasks
```python
# app/tasks.py
import logging
from celery import shared_task
from django.utils import timezone

logger = logging.getLogger(__name__)

@shared_task(
    bind=True,
    max_retries=3,
    default_retry_delay=60,
    queue='api_send',          # match app.conf.task_queues in cansystem/celery.py
)
def send_platform_orders_to_external_api(self):
    """Send pending orders to external API. Retries up to 3× on failure."""
    try:
        # business logic here
        logger.info("send_platform_orders_to_external_api started at %s", timezone.now())
    except Exception as exc:
        logger.exception("send_platform_orders_to_external_api failed: %s", exc)
        raise self.retry(exc=exc)
```

### Queue Routing (cansystem/celery.py)
```
default      → lightweight, frequent tasks
heavy_tasks  → platform sync (Shopee, TikTok) — long-running
api_send     → external API calls & VBIS Cloud sync — time-sensitive
```
Always declare the correct `queue` in `app.conf.task_routes` **and** in the `@shared_task(queue=...)` decorator.

## Exception Handling
```python
# app/exceptions.py
from rest_framework.views import exception_handler
from rest_framework.response import Response

def custom_exception_handler(exc, context):
    response = exception_handler(exc, context)
    if response is not None:
        response.data = {
            'success': False,
            'error': {
                'code':    getattr(exc, 'default_code', 'ERROR'),
                'message': str(exc.detail) if hasattr(exc, 'detail') else str(exc),
            }
        }
    return response
```

Register in `settings.py`:
```python
REST_FRAMEWORK = {
    'EXCEPTION_HANDLER': 'app.exceptions.custom_exception_handler',
    ...
}
```

## URL Registration
```python
# api/urls.py
from rest_framework.routers import DefaultRouter
from app.views import OrderViewSet

router = DefaultRouter()
router.register(r'orders', OrderViewSet, basename='order')

urlpatterns = router.urls
```

## Logging Best Practice
```python
import logging
logger = logging.getLogger(__name__)

# Use structured context wherever possible
logger.info("Order sent successfully", extra={"order_id": order.id, "platform": order.platform})
logger.exception("Unexpected error processing order %s", order.id)
```

## Security Rules
- Never put secrets in code — use `python-dotenv` / environment variables
- Always use Django ORM (no raw SQL unless absolutely required; if raw SQL, use params)
- `DEBUG = False` in production
- Set `ALLOWED_HOSTS` and `CORS_ALLOWED_ORIGINS` explicitly
- Use `IsAuthenticated` as the default permission class in DRF settings

## Migrations Checklist
```bash
python manage.py makemigrations --check   # verify no undetected changes
python manage.py makemigrations
python manage.py migrate
```
- Never edit an applied migration — create a new one
- Add `db_index=True` or `Meta.indexes` for frequently queried fields

## Checklist Before Every PR
- [ ] PEP 8 compliant — no lines > 120 chars
- [ ] All inputs validated in serializer (`validate_<field>` or `validate()`)
- [ ] Permissions declared on every view
- [ ] No secrets in code, all via environment variables
- [ ] New models have migrations
- [ ] Celery tasks assign the correct `queue`
- [ ] Errors logged with `logger.exception()` (not `print`)
- [ ] `@extend_schema` added for drf-spectacular docs
- [ ] Tests written: unit for services, integration for views
