Django Debug Toolbar doesn't work for views that return JSON - it needs an html page to add itself.

But we would like to use it anyway.

Here is a nice hack from [SO](https://stackoverflow.com/a/19249559/481815)

```python
class NonHtmlDebugToolbarMiddleware(object):

    @staticmethod
    def process_response(request, response):
        if request.GET.get('debug') == '':
            if response['Content-Type'] == 'application/octet-stream':
                new_content = '<html><body>Binary Data, ' \
                    'Length: {}</body></html>'.format(len(response.content))
                response = HttpResponse(new_content)
            elif response['Content-Type'] != 'text/html':
                content = response.content
                response = HttpResponse('<html><body><pre>' + content + '</pre></body></html>')

        return response
```
