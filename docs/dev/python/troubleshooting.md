# Troubleshooting

### &#x20;pytesseract

Install `tesseract-data-eng`:

```
sudo pacman -S tesseract-data-eng
```

### Extra character

Sometimes, when using strange Python libraries to extract text from a special file such as an image, special characters such as %0A (line feed) may be added. To remove them, we can use the regex library with a white list of allowed characters:

```python
import re

text = re.sub("[^A-Za-z0-9]+", "", text)
```
