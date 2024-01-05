# WebScraping-databaseCreation
## In this portfolio project I scraped a website using Python. I downloaded the image data and saved all scraped data in a json file.


### 1. Import Libraries
```ruby
from bs4 import BeautifulSoup
import requests
import json
import os
import html
from PIL import Image
```

### 2. Get the HTML content and extract information
```ruby
html_text = requests.get('https://innodis.mu/innodis/pages/modules/product/allproducts.php').text

soup = BeautifulSoup(html_text, 'html.parser')

products = soup.find_all('figure')
```

### 3. Define the file path to save the JSON data - Product
```
file_path_json = 'C:/Users/MRT/DataAnalystProjects/innodis/product_innodis.json'

existing_data = []

if os.path.exists(file_path_json) and os.path.getsize(file_path_json) > 0:
   # Load existing data from the JSON file, if it exists and is not empty
   with open(file_path_json, 'r') as file:
       existing_data = json.load(file)
```

### 4. Create a loop to extract desired information from the website.
```ruby
for product in products:
        
    product_id = product.find('input', class_ = 'form-control')['value']
    product_name = html.unescape(product.span.text)
    product_category = product.find('input', id = 'category02'+product_id)['value']
    product_img_url = product.find('img', class_ = 'content-image')['src']
    product_img = "static/products/"+product_id+".jpg"
    product_price_string = (product.strong.text.split()[-1].replace(",",""))
    
    if product_price_string == "******":
       product_price = 0.00
    else:
       product_price = (format(float(product_price_string), ".2f"))

    # Define the local file path to save the image
    file_path = 'C:/Users/MRT/DataAnalystProjects/innodis/products/'+product_id+'.jpg'
    
    # Send an HTTP request to the image URL and save the response content to the file
    response = requests.get(product_img_url)
    if response.status_code == 200:
       with open(file_path, 'wb') as file:
           file.write(response.content)
           print("Image downloaded successfully.")
    else:
       print("Failed to download the image.")

    data = {
       "model": "shop.product",
       "pk": int(product_id),
       "fields": {
           "name": product_name,
           "category": int(product_category),
           "business": "Innodis",
           "display": False,
           "description": "",
           "price": product_price,
           "img": product_img,
           "prompt_texts": ""
       },
   }

    # Append the new product data to the existing data
    existing_data.append(data)

    # Save the updated data to the JSON file
    with open(file_path_json, 'w') as file:
       json.dump(existing_data, file, indent=4)

    # print(f"Product with ID {product_id} inserted into product_innodis.json.")

print("All product data inserted into product_innodis.json.")
```

### 5. Resize images
```ruby
extensions = ['jpg', 'jpeg', 'png']
files = os.listdir("C:/Users/MRT/DataAnalystProjects/innodis/products/")

for file in files:
   ext = file.split(".")[-1]
   filename = file.split(".")[0]
   # print(ext)
   if ext in extensions:
       try:
           im = Image.open("C:/Users/MRT/DataAnalystProjects/innodis/products/"+file)
           im_resized = im.resize((200,200))
           filepath = f"C:/Users/MRT/DataAnalystProjects/innodis/products_resized/{filename}-200-200.{ext}"
           im_resized.save(filepath)
       except OSError as e:
           print(f"Error processing file: {file}. Reason: {e}")
```

### 6. Define the file path to save the JSON data - Product Category
```ruby
categories = soup.find_all('li', class_ = 'dropdown')

# Define the file path to save the JSON data
file_path_json = 'C:/Users/MRT/DataAnalystProjects/innodis/product_category_innodis.json'

existing_data = []

if os.path.exists(file_path_json) and os.path.getsize(file_path_json) > 0:
   # Load existing data from the JSON file, if it exists and is not empty
   with open(file_path_json, 'r') as file:
       existing_data = json.load(file)

for category in categories:  
   category_name = category.text.replace('\n',"")
  
   category_a = category.a
   if category_a:
       category_url = category.a['href']     
   else:
       print("No 'a' tag found for category:", category_name)

   if category_url and 'id=' in category_url:
       category_id = category_url.split('=')[-1]
       
       data = {
           "model": "shop.productcategory",
           "pk": int(category_id),
           "fields": {
               "name": category_name,
               "parent": "null",
               "description": "",
               "display": False,
               "img": "/static/products/blank.png",
               "business": 10,
               "business_only": True,
               "prompt_texts": ""
           },
       }
       
       # Append the new product data to the existing data
       existing_data.append(data)

       # Save the updated data to the JSON file
       with open(file_path_json, 'w') as file:
           json.dump(existing_data, file, indent=4)

       # print(f"category with ID {category_id} inserted into product_innodis.json.")

   print("All category data inserted into product_category_innodis.json.")
```

### The JSON data looks like below :
![image](https://github.com/sheeksha/WebScraping-databaseCreation/assets/69764380/37a763c7-e859-4672-af8e-42b7c3a11ef1)

![image](https://github.com/sheeksha/WebScraping-databaseCreation/assets/69764380/46524d3d-061f-477b-8be4-be764ef74ef4)

