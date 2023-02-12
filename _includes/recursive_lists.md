{% for item in include.list %}{% if item[0] %}{{ include.prefix }}Sublist:
{% assign p = '  ' | append: include.prefix %}{% include recursive_lists.md list=item prefix=p %}{% else %}{{ include.prefix }}{{ item }}
{% endif %}{%- endfor -%}
