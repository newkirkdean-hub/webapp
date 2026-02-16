# Original CSS Styles

## Overview
This CSS file contains the base styling for the web application, including layout, typography, and button styles.

---

## CSS Code

```css
.main_layout {
    width: 100%;
    height: 100vh;
    display: flex;
    flex-direction: column;
    position: relative;
    overflow: visible; /* Ensure the dropdown isn't clipped */
    background-color: #993300;
}

body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    background-color: #993300;
    overflow-y: hidden;
}


.lable-text {
    color: #FFFFFF; /* Makes the text white */    
    font-weight: bold;
}

/*= = = =BUTTON STUFF = = = */
.grid-button {
    color: #FFFFFF;
    width: 90%;
    height: 65px;
    display: block;
    text-align: center;
    font-size: 16px;
    border: none;
    font-weight: bold;
    background-color: #006666;
    cursor: pointer;
    border-radius: 8px;
}

.grid-button:hover {
    background-color: #007979;
}
```

---

## Style Breakdown

### Main Layout (`.main_layout`)
- **Purpose**: Primary container for the application
- **Display**: Flexbox with column direction
- **Dimensions**: Full width and viewport height
- **Background**: Brown/rust color (#993300)
- **Overflow**: Visible to prevent dropdown clipping

### Body Styles
- **Font**: Arial, sans-serif
- **Reset**: No margins or padding
- **Box Model**: Border-box sizing
- **Background**: Matches main layout (#993300)
- **Scroll**: Vertical scrolling disabled

### Label Text (`.lable-text`)
- **Color**: White (#FFFFFF)
- **Weight**: Bold
- **Note**: Class name has typo ("lable" instead of "label")

### Grid Button (`.grid-button`)
| Property | Value | Description |
|----------|-------|-------------|
| Color | #FFFFFF | White text |
| Width | 90% | Responsive width |
| Height | 65px | Fixed height |
| Background | #006666 | Teal color |
| Hover Background | #007979 | Lighter teal |
| Border Radius | 8px | Rounded corners |
| Font Size | 16px | Standard readable size |
| Font Weight | Bold | Emphasized text |
| Cursor | Pointer | Interactive indicator |

---

## Color Palette

| Color Code | Usage | Description |
|------------|-------|-------------|
| `#993300` | Background | Dark rust/brown |
| `#FFFFFF` | Text | White |
| `#006666` | Button | Teal (default) |
| `#007979` | Button Hover | Lighter teal |

---

## Notes

- Consistent color scheme throughout (rust brown background with teal accent buttons)
- Buttons are responsive (90% width) and highly visible
- Flexbox layout for modern responsive design
- Minor typo in class name: `.lable-text` should be `.label-text`
- Body prevents vertical scrolling (`overflow-y: hidden`)

---

## Repository Information
- **File**: originalCss.txt → originalCss.md
- **Repository**: newkirkdean-hub/webapp
- **Branch**: main
