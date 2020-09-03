# Features

- Keyword repartition
- Length content 
- Check title article length
- Url is optimized
- Number internal Links 

# Installation

## Installing the application in Django

To use this application, you need first to add it to your config file.

In your config file (ex: settings.py):

```
# Application definition

INSTALLED_APPS = [
	...
	'seo'
	...
]
```

## SEO Pannel

To setup the keyword for SEO, you need to add a special SEO Pannel that will appear in your page creation in wagtail admin.

The module contains a model in models/SeoPage, the model need to be used as a base for your page models.

Example of your model:

``` 
class HomePage(SeoPage):
    date = models.DateField("Post date")
    intro = models.CharField(max_length=250)
    delay = models.IntegerField(default=0, validators=[MaxValueValidator(99), MinValueValidator(0)])
    body = StreamField([
        ('text', RichTextBlock(blank=True, features=['h2', 'h3', 'h4', 'bold', 'italic', 'link',
                                                     'code', 'ol', 'ul', 'hr', 'document-link', 'image', 'embed', 'superscript', 'subscript', 'strikethrough', 'blockquote'])),
        ('rawHtml', RawHTMLBlock(blank=True)),
    ], blank=True)
    images_keyword = models.CharField(max_length=250, blank=True)
    selected_image = models.ForeignKey(
        'wagtailimages.Image',
        null=True,
        blank=True,
        on_delete=models.SET_NULL,
        related_name='+'
    )

    keep_slug = models.BooleanField(
        verbose_name=('Keep current slug'),
        default=False,
        help_text=("Keep current slug or save to generate a new slug.")
    )

    def _get_autogenerated_slug(self, base_slug):
        """Redefinition of wagtail's _get_autogenerated_slug so you can use your own slug generator."""
        return self.slug

    search_fields = Page.search_fields + [
        index.SearchField('intro'),
    ]

    content_panels = Page.content_panels + [
        MultiFieldPanel([
            FieldPanel('date'),
            FieldRowPanel([
                FieldPanel('delay'),
            ]),
        ], heading="Blog information"),
        FieldPanel('intro'),
        StreamFieldPanel('body'),
        FieldRowPanel([
            FieldPanel('images_keyword'),
        ], heading="Images"),
        ImageChooserPanel(field_name="selected_image", heading="Image sélectionnée"),
    ]

    promote_panels = [
        MultiFieldPanel([
            FieldPanel('slug'),
            FieldPanel('keep_slug'),
            FieldPanel('seo_title'),
            FieldPanel('show_in_menus'),
            FieldPanel('search_description'),
        ], heading="Common Page Configuration"),
    ]

    edit_handler = TabbedInterface([
        ObjectList(content_panels, heading='Content'),
        ObjectList(promote_panels, heading="Promote"),
        SeoPage.seo_object_list,
        ObjectList(Page.settings_panels, heading='Settings')
    ])
```

## Routing

In your routing projet file `urls.py`
```
from django.conf.urls import url
from django.urls import include

urlpatterns = [
    ...
    url(r'^seo/', include('seo.urls'), name='seo'),
]
```

## DB Migration

Now you can detect the change
`python manage.py makemigrations`

And apply it on DB
`python manage.py migration`


## Test

`pytest`