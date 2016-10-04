# mypy-django
## Type stubs to use the mypy static type-checker with your Django projects

This project includes the [PEP-484](https://www.python.org/dev/peps/pep-0484/) compatible "type stubs" for Django APIs. Using a compliant checking tool (typically, [mypy](http://mypy-lang.org/)), it allows you to document and verify more of your code. Your annotated code will look like:

```python
def vote(request: HttpRequest, question_id: str) -> HttpResponse:
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        return render(request, 'polls/detail.html', {'question': question})
    else:
        selected_choice.votes += 1
        selected_choice.save()
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```

If you use incorrect annotations, like in the following example

```python
class ResultsView(generic.DetailView):
    model = Question

    def get_template_names(self) -> str:
        if some_condition():
            return "template_a.html"
        else:
            return "template_b.html"
```

Running mypy will report the problem:

```
$ mypy --strict-optional -p polls
...
polls/views.py: note: In class "ResultsView":
polls/views.py:41: error: Return type of "get_template_names" incompatible with supertype "SingleObjectTemplateResponseMixin"
polls/views.py:41: error: Return type of "get_template_names" incompatible with supertype "TemplateResponseMixin"

```

## Installation and usage

You'll need to install mypy (other PEP-484 checkers might work, but I haven't tested them).
`pip install mypy-lang` should do the trick. There are no other requirements.

This is not a python package (no actual executable code), so this is not installed with `pip` or
available in [PyPI](https://pypi.python.org/pypi). You can just `git clone` the latest version from
https://github.com/machinalis/mypy-django.git or download and unzip https://github.com/machinalis/mypy-django/archive/master.zip

Once you have your copy, set your `MYPYPATH` environment variable to point to the files. For example (in Linux/bash):

```
$ export MYPYPATH=/home/dmoisset/mypy-django/
$ ls $MYPYPATH
django
$ ls $MYPYPATH/django
conf  core  http  __init__.pyi  urls  utils  views

```

If you don't see the above (the second line might have a few more items in your computer), check
that the path exists, and that it points to the correct level in the directory tree.

## Motivation

We are building this as a tool at [Machinalis](http://www.machinalis.com) to improve the quality of the Django projects we build for our clients. Feel free to contact me if you want to hear more about
how we use it or how it can be applied. I can be found at dmoisset@machinalis.com or at [@dmoisset](http://twitter.com/dmoisset) via
Twitter.

In a more general perspective, it makes sense to use static typing for Django given the following:

1. Much of the user application code for Django projects consists in operating on objects defined
   by the framework. Unlike other APIs where you mostly pass around standard python data structures,
   this means that you don't get much benefit from PEP-484 static typing because everything gets
   annotated as `Any` (i.e. unchecked)
2. A large part of the framework follows a very structured, almost declarative approach where you
   just fill-out a structure (for example, defining models, admin options, generic views, url
   routers, forms, settings)
3. Django already has a policy of checking types before starting serving. Many of the system checks
   performed by `manage.py check` are actually type checks. So this fits very well with the
   framework philosophy

## Full example

I reimplemented most of the standard Django tutorial with annotations, so you can see how it
looks. The code (and a README with some details of problems and solutions found when annotating) are available at
https://github.com/machinalis/mypy-django-example

## Known issues

* The current version is mainly focused on supporting Django 1.10 under python 3.x. Given that the
APIs I cover are the core components and haven't changed much, you probably can work with older
versions of Django and it might work. Python 2.x will not be supported (The code uses `str` to
describe arguments/return values that can be text strings, i.e. unicode).
* Many django modules that you might import are not supported yet. So you might need to silence
with `# type: ignore` some messages like:
```
polls/views.py:1: error: No library stub file for module 'django.db.models.query'
```
* It's recommended that you run mypy with the `--strict-optional` option; many of the stubs assume
that you do, and you might get some warnings inside the stub files if you don't use it.

## Roadmap

### v0.1 - Initial release - October 2016

* Request and Response objects
    * Including supporting classes like QueryDict and file objects
* Generic views
* URL resolver
* Other miscellaneous components required by the above (timezones, cookies, ...)

### v0.2 - In development

* Admin support
* django.shortcuts
* Paginators

### Probably never

* Querysets may have some partial support, but complex arguments (like the ones for `filter` and
`get` queries) or `Q` and `F` objects are beyond the expressive possibilities of mypy as it is now.
* The template language is a separate language and can not be covered by mypy, so any type errors
inside the template can not be detected by it.

## License

BSD. See LICENSE file for details
