# Use for ... empty in Django templates.


```
{% if books %}
  {% for book in books %}
    <li>{{ book }}</li>
  {% endfor %}
{% else %}
  <li>Sorry, there are no books.</li>
{% endif %}
```

```
{% for book in books %}
  <li>{{ book }}</li>
{% empty %}
  <li>Sorry, there are no books.</li>
{% endfor %}
```

The first block took 0.9ms and second block took 0.8ms which means it is 11% faster than previous code



# Make Django Development Server Persistent.


```
$ while true; do python manage.py runserver; sleep 4; done
```

Shell script

```
while true; do
  echo "Re-starting Django runserver"
  python manage.py runserver
  sleep 4
done
```




# Hyperlinking foreignkeys in admin

```
class ModelAdminWithForeignKeyLinksMetaclass(RenameBaseModelAdminMethods):

    def __new__(cls, name, bases, attrs):
        new_class = super(ModelAdminWithForeignKeyLinksMetaclass, cls).__new__(cls, name, bases, attrs)

        def foreign_key_link(instance, field):
            target = getattr(instance, field)
            return u'<a href="../../%s/%s/%s/">%s</a>' % (
                target._meta.app_label, target._meta.module_name, target.pk, unicode(target))

        for col in new_class.list_display:
            if col[:8] == 'link_to_':
                field_name = col[8:]
                method = partial(foreign_key_link, field=field_name)
                method.__name__ = col[8:]
                method.allow_tags = True
                method.admin_order_field = field_name
                setattr(new_class, col, method)

        return new_class
```

Instead of moving back & forth, now we can directly navigate.



# Loading urls/static files


```
# urls.py

urlpatterns = (
  '',
  url(r'^django', django_view, name='django-view'),
  url(r'^django-plus', django_view_plus, name='django-view-plus'),
)
```

```
# templates
templates
├── index.html
├── 404.html
│   ├── index.html
│   ├── base.html

# views.py 
def foo(request):
    return render(request, 'index.html')

def bar(request):
    return render(request, 'index.html')
```

First Match --> First Serve

Correctly Order Urls/Files

```
# urls.py

urlpatterns = (
  '',
  url(r'^django-plus', django_view_plus, name='django-view-plus'),
  url(r'^django', django_view, name='django-view'),
)
```

```
# templates
templates
├── index.html
├── 404.html
│   ├── index.html
│   ├── base.html

# views.py 
def foo(request):
    return render(request,  'index.html')

def bar(request):
    return render(request, 'bar/index.html')
```



# Setting Initial Values In Forms


```
# forms.py

class ContactForm(forms.Form):
   
  def __init__(self, *args, ##kwargs):
      super().__init__(*args, ##kwargs)
      self.fields['name'].initial =  'override'
  
  name = forms.CharField(max_length=10) 
```

Won't Work If View Adds Initials

```
# views.py

form = AdvancedForm(initial={'name': 'precedence'})


# forms.py

class ContactForm(forms.Form):

 def __init__(self, *args, **kwargs):
      super().__init__(*args, **kwargs)
      self.fields['name'].initial = 'override'  # gotcha
```

Set Initial Values Correctly

```
# forms.py 

class ContacdForm(forms.Form):
   
   def __init__(self, *args, **kwargs):
      super().__init__(*args, **kwargs)
      self.initial['name'] = 'override'  # !!!  
   
   name = forms.CharField(max_length=10) 
```


# Add Two Forms In Same Page

```
# views.py

if request.method == 'GET':
    form1 = Contactform1()
    form2 = Contactform2()

if request.method == 'POST':
    form1 = Contactform1(request.POST)
     if form1.is_valid():
          # ...
     else:
          # ...

    form2 = Contactform1(request.POST)
     if form2.is_valid():
          # ...
     else:
          # ...
```


# Issues with validation


```
# views.py

if request.method == 'POST':
     form1 = Contactform1(request.POST)
     if form1.is_valid():
          # ...
     else:
          # ...

    form2 = Contactform1(request.POST)
     if form2.is_valid():
          # ...
```


Use Different Action For Urls 

```
<!-- template.html ->

<form method="POST" action="/url1/">
</form>

<form method="POST" action="/url2/">
</form>

```

Alternatively Use Different Button Names

```
<!-- template.html ->

<form>
   <button type="submit" name="form1">Submit</button>
</form>

<form>
   <button type="submit" name="form2">Submit</button>
</form>
```


```
# views.py

if 'form1' in request.POST:
      # do action1

if 'form2' in request.POST:
      # do action2
```


# Create Nullable Field In Database


```
# models.py

class Book(models.Model):
    count = models.IntegerField(max_length=10, null=True)
```

Form Doesn't Accept Empty Values

Add blank=True

```
# models.py

class Book(models.Model):
    count = models.IntegerField(max_length=10, null=True, blank=True)
```



# Admin List Display


```
# admin.py

class BookAdmin(BaseAdmin):
    list_display = ('name') + BaseAdmin.list_display
```

Django Is Expecting A Tuple/Iterable

```
# admin.py

class BookAdmin(BaseAdmin):
    list_display = ('name') + BaseAdmin.list_display  #  gotcha
```

Similar issues with urls

```
# urls.py

urlpatterns = (
    url(r'^$', index)  # gotcha
)
```



Pass A Tuple/Iterable

```
# admin.py

class BookAdmin(BaseAdmin):
    list_display = ('name',) + BaseAdmin.list_display  #  gotcha


# urls.py

urlpatterns = (
    url(r'^$', index),
)
```


# Generic Views


```
# views.py

from django.views.generic.edit import DeleteView

class AuthorDelete(DeleteView):
    model = Author
    success_url = reverse('author-list')
```


Url not available

Functions - body is executed when function is called

Class - Variables are evaluated immediately


Lazy load url

```
# views.py

from django.views.generic.edit import DeleteView
from django.core.urlresolvers import reverse_lazy

class AuthorDelete(DeleteView):
    model = Author
    success_url = reverse_lazy('author-list')
```

Alternatively Override Get Success Url

```
# views.py

from django.views.generic.edit import DeleteView

class AuthorDelete(DeleteView):
    model = Author

   def get_success_url(self):
        return reverse('author-list')
```



# Get Last Item In A Query Set


```
# shell 
In [2]: Proposal.objects.filter()
Out[2]: [<Proposal: Consectetur Proposal>, <Proposal: Rem Sit Proposal>]

In [3]: p = Proposal.objects.filter()

In [4]: p[-1]
```


Django Querysets Are Not Lists


```
---------------------------------------------------------------------
AssertionError                      Traceback (most recent call last)
<ipython-input-14-f4039f8a3e39> in <module>()
----> 1 p[-1]

AssertionError: Negative indexing is not supported.


In [6]: type(p)
Out[6]: django.db.models.query.QuerySet
```

Don't Compare Queryset With Empty List

```
In [9]: Proposal.objects.filter(id=1111111111111)
Out[9]: []

In [10]: p == []
Out[10]: False
```

Use list() or .count() 

```
# convert to list 
In [4]: pp = list(p)

In [5]: pp[-1]
Out[5]: <Proposal: Consectetur Proposal>

# use count and then substract
In [15]: p = Proposal.objects.filter(id=1)

In [16]: p[p.count()-1]
Out[16]: <Proposal: Consectetur Proposal>

```


# Executing Raw SQL


```
# get books that begin with `a` 
from django.db import connection

query = "SELECT * FROM gutenberg.books LIKE 'a%';"

with connection.cursor() as c:
    c.execute(query)
```

Throws TypeError

```
# file.py

# get books that begin with `a` 

query = "SELECT * FROM gutenberg.books LIKE 'a%';"  # gotcha

with connection.cursor() as c:
    c.execute(query)
```

% is a placeholder in query string


Use `%` To Escape `%`

```
# file.py

# get books that begin with `a` 

query = "SELECT * FROM gutenberg.books LIKE 'a%%';"

with connection.cursor() as c:
    c.execute(query)
```


# Select distinct(*fields) On A Model


```
# shell

>>> Books.objects.order_by('author').distinct('author')
>>> Books.objects.order_by('author', 'date').distinct('author', 'date')
```

Works Only On PostgreSQL

Use Same db For All purposes


# Get Items Related To Logged In User


```
#  models.py
from django.contrib.auth.models import User

class PrivilegedUser(models.Model):
    vip = models.Foreignkey('User')

# views.py

from .models import PrivilegedUser

def _validate_user(user):
    return PrivilegedUser.objects.filter(vip=user).exists()


def home_view(request):
    vip = _validate_user(request.user)
```


TypeError: User Objects Are Lazy


Wake Lazy Objects

Authenticate users or do attribute access

```
#  models.py
from django.contrib.auth.models import User

class PrivilegedUser(models.Model):
    vip = models.Foreignkey('User')


# views.py

from .models import PrivilegedUser

def _validate_user(user):
    return user.is_authenticated() and 
       PrivilegedUser.objects.filter(vip=user).exists()

def home_view(request):
    vip = _validate_user(request.user)
```

