# Chat Project
### Setup Instructions
### 1.Create and activate virtual environment
'''
python -m venv venv
source venv/bin/activate   # On Windows: venv\Scripts\activate

'''

### 2.Install required packages
'''
pip install Django channels channels_redis
'''
Also make sure Redis is running locally. You can start it via:
'''
redis-server
'''

### 3.Project & App Initialization
'''
django-admin startproject chat_project
cd chat_project
python manage.py startapp chat
'''

### 4.Configure '''settings.py'''
In '''chat_project/settings.py''':
'''
INSTALLED_APPS = [
    ...,
    'channels',
    'chat',
]

ASGI_APPLICATION = 'chat_project.asgi.application'

CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {
            "hosts": [('127.0.0.1', 6379)],
        },
    },
}
'''

### 5.ASGI config ('''chat_project/asgi.py''')
'''
import os
from channels.auth import AuthMiddlewareStack
from channels.routing import ProtocolTypeRouter, URLRouter
from django.core.asgi import get_asgi_application
import chat.routing

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'chat_project.settings')

application = ProtocolTypeRouter({
    "http": get_asgi_application(),
    "websocket": AuthMiddlewareStack(
        URLRouter(chat.routing.websocket_urlpatterns)
    ),
})
'''


### 6.WebSocket Routing ('''chat/routing.py''')
'''
from django.urls import re_path
from . import consumers

websocket_urlpatterns = [
    re_path(r'ws/chat/(?P<room_name>\w+)/$', consumers.ChatConsumer.as_asgi()),
]
'''

### 7.Views and URLs
'''chat/views.py'''
'''
from django.shortcuts import render

def room(request, room_name):
    return render(request, 'chat/room.html', {'room_name': room_name})
'''
'''chat/urls.py'''
'''
from django.urls import path
from . import views

urlpatterns = [
    path('<str:room_name>/', views.room, name='room'),
]
'''
Add to '''chat_project/urls.py'''
'''
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('chat/', include('chat.urls')),
]
'''

### 8.Templates Setting
In '''settings.py''', ensure this is present:
'''
TEMPLATES = [
    {
        ...
        'DIRS': [BASE_DIR / 'chat/templates'],
        ...
    },
]
'''

### 9.Migrate and Run
'''
python manage.py migrate
python manage.py runserver
'''
Then go to:
'''
http://127.0.0.1:8000/chat/lobby/
'''
