# PHP function to minify CSS

I'll create a PHP function to minify CSS. This will strip out unnecessary whitespace, comments, and newlines to reduce file size.

```php
<?php
/**
 * CSS Minifier function
 * 
 * This function removes comments, whitespace, and unnecessary characters
 * from CSS to reduce file size
 * 
 * @param string $css The CSS content to minify
 * @return string The minified CSS
 */
function minify_css($css) {
    // Remove comments
    $css = preg_replace('!/\*[^*]*\*+([^/][^*]*\*+)*/!', '', $css);
    
    // Remove whitespace
    $css = preg_replace('/\s+/', ' ', $css);
    
    // Remove whitespace around special characters
    $css = preg_replace('/\s*([\{\};:,>+])\s*/', '$1', $css);
    $css = preg_replace('/([\{\}:;,>+])\s+/', '$1', $css);
    
    // Remove trailing semicolons
    $css = preg_replace('/;}/', '}', $css);
    
    // Remove leading zeros from decimal values
    $css = preg_replace('/0(\.\d+)/', '$1', $css);
    
    // Remove units from zero values
    $css = preg_replace('/(^|[^0-9])0+(px|em|rem|%|in|cm|mm|pc|pt|ex|vh|vw|vmin|vmax)/', '${1}0', $css);
    
    // Shorten hex colors when possible (#ffffff to #fff)
    $css = preg_replace('/#([a-f0-9])\1([a-f0-9])\2([a-f0-9])\3/i', '#$1$2$3', $css);
    
    return trim($css);
}

/**
 * Example usage to minify a CSS file
 */
function minify_css_file($input_file, $output_file = null) {
    if (!file_exists($input_file)) {
        return false;
    }
    
    $css = file_get_contents($input_file);
    $minified = minify_css($css);
    
    if ($output_file) {
        file_put_contents($output_file, $minified);
        return true;
    } else {
        return $minified;
    }
}

// Example usage:
// $minified = minify_css_file('style.css', 'style.min.css');
// or
// $css = file_get_contents('style.css');
// $minified = minify_css($css);
?>

```

Here's how to use the function:

To minify CSS from a string:
```php
$css = "body {
    font-family: Arial, sans-serif;
    color: #333333;
    margin: 0;
    padding: 0;
}";

$minified = minify_css($css);
echo $minified;
// Outputs: body{font-family:Arial,sans-serif;color:#333;margin:0;padding:0}
```

To minify a CSS file:
```php
minify_css_file('path/to/style.css', 'path/to/style.min.css');
```

The minification process:
1. Removes all comments
2. Eliminates extra whitespace
3. Removes whitespace around special characters
4. Removes unnecessary semicolons
5. Shortens hex color codes where possible
6. Removes leading zeros from decimal values
7. Removes units from zero values

This should significantly reduce your CSS file size while maintaining functionality.