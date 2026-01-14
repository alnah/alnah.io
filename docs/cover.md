# Cover Images

Guidelines for blog post cover images using the Hugo Stack theme.

## Recommended Specifications

| Property    | Value                    |
| ----------- | ------------------------ |
| Dimensions  | 1200 x 630 px            |
| Aspect      | 16:9 or 3:2 (landscape)  |
| Format      | JPG or WebP              |
| Max size    | ~500 KB                  |

## Theme Behavior

Stack uses `object-fit: cover` which crops images to fit containers:

**Homepage cards:**
- Mobile: 150px height
- Tablet: 200px height
- Desktop: 250px height

**Article page:**
- Max height: 50vh (50% viewport height)
- Width: 100%

## Usage

Place `cover.jpg` in the post bundle and reference in front matter:

```yaml
---
title: "Post Title"
image: cover.jpg
---
```

## Tips

- Use landscape orientation to minimize cropping
- Center important content (faces, text) in the middle third
- 1200x630 doubles as Open Graph image for social sharing
