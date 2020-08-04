One day I got tired of excessive whitespace in html produced by rendering templates.

Jinja2 has those `\{%- foo -%}` thingies, but regular django templates don't. And they're annoying anyway.

So I did this:

```python
class RemoveWhitespace(MiddlewareMixin):
    def process_response(self, request, response, _ws=re.compile(b'\n[ \t\r\n]+').sub):
        if not response.has_header('Content-Encoding') and \
                response.has_header('Content-Type') and \
                response['Content-Type'].startswith('text/html'):
            response.content = _ws(b'\n', response.content)
        return response
```
