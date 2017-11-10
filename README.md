# Discoverings

The images that Product Details Page displays can be divided among 4 types. For the product images we have the **Product Main Image and Alternative Angles** and the **Product Images With Other Colors** for any product that has more than one color. For the color swatch (the images where the user clicks to choose the color) we have **Solid Colors** or **Images as Colors**.

For the following sessions, please consider `URL` as a combination of server URL and banner. For example:
- If server URL is `s7d9.scene7.com/`,
- And banner is `Lord and Taylor`,
- Then `URL` will be `s7d9.scene7.com/LordandTaylor/`.

Also, the other two important components of the following are the `product_code` and the `upc`, both are proprieties of a product, but `product_code` refers to the product in general, while `upc` refers to a specific product, considering its variations, such as color and size.

## Product Main Image and Alternative Images
The Product Main Image is the image that appears when the PDP of a product is shown. There may or not be Alternative Images for the product, such as other angles or a closer view of it.

Images come from `s7d9.scene7.com/` server.

### Current The Bay/La Baie Website
Product main image URL is:
```
URL/{upc}_main
```
and the alternative images follow a pattern where instead of `_main` we have `_alt1`, `_alt2`, and so on for as many alternative images the product has.

### Common Platform
Product main image URL is:
```
URL/{product_code}
```
and the URLs for the alternative angles come from a JSON returned by:
```
URL/{product_code}_IS?req=set,json,UTF-8&labelkey=label&id=93821123&handler=s7sdkJSONResponse
```
The URL of these images follow a pattern where after the product code we have only `_A1`, `_A2`, `_A3`, and so on. Those `query parameters` are important, without them this URL would only return the main image.

### Our Approach
We tried to make the code in the common platform form such an URL that would be the same as the current The Bay website, but the way PDS gets the alternative images breaks the `canvas` in which all product images appear. To keep the images in the same canvas and using the same URL the current website uses, we would need to make `s7d9.scene7.com/TheBay/{upc}_IS` return URLs with the `_alt` instead of the `_A` in the end. We still don't know how to access or change anything in this server.

## Product Images With Other Colors (Colorized)
These are the images of the product with its other colors applied to it. There are products that have no color, and for those the color swatch (component to choose among the color options) will not appear and the only image visible will be the main image. There are products with a single color, for them the color swatch **will appear**, and if clicked the image might change its URL.

Images come from `s7d9.scene7.com/` server

### Current The Bay/La Baie Website
URL is:
```
URL/{upc}_main
```
Even thought this URL is the same as for the main product image, it returns the colorized image because the `upc` already refers to a specific color (and size), so the `upc` for the main product image will be different than for any colorized product image.

### Common Platform
URL is:
```
URL/{product_code}_{COLORNAME}
```
This URL is partially mounted in Product Detail Service, but this approach proved to be a problem because the color name in the color swatch is something that has to be internationalized, while the `COLORNAME` in the URL should not be (for the URL to have an image), but both of them derive from the same attribute of a product.
So, for example, when we access the PDP in french and a product has the color `Blue`, the name of the color in the product attributes is `Blue` but is internationalized to `Bleu`, so the image URL ends up as `URL/{product_code}_BLEU` instead of `URL/{product_code}_BLUE`. This causes the image to not be found, and because this internationalization occurs in Product Service (PS), Product Detail Service (that consumes from PS) does not see the original color names.

### Our Approach
By using the banner for The Bay and manually setting the product `upc` for all variants (different color and sizes) in Mongo to point to an image that exists in the current website (and therefore the URL does not give a not found error) we managed to see that PDS was returning values that were meant to find the right URLs, but as said before, not having a way to deal with the JSON returned with the alternative images broke the canvas, and we couldn't see the actual images or their URLs in place when we clicked in the colors of the color swatch.

## Color Swatch With Images as Color
Images come from `image.s5a.com` server.

### Current The Bay/La Baie Website
URL is:
```
URL/{upc}_swatch
```

### Common Platform
URL is:
```
URL/{upc}
```

### Our Approach
Some variants (attributes that refer to color, size and other specifications of a product) have the attribute `is_color_an_image` as true, and for these instead of looking for a Hex code to choose a solid color for the swatch, the color appears as an image.
For changing the way the common platform deals with this to the way the current website does, we would have to change the way this URL is produced as well.
