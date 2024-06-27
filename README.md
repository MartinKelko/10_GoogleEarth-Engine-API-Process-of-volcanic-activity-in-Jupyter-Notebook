# Fernandina Volcano Lava Flow Visualization

## Description
This code demonstrates a comprehensive workflow using the Google Earth Engine API in Python to visualize the lava flow at Fernandina volcano on the Galapagos Islands. It integrates data filtering, image processing, visualization parameter adjustment, and interactive display within the Jupyter Notebook environment, enabling detailed exploration and analysis of satellite imagery captured by Sentinel-2. Adjustments in visualization parameters, such as gamma correction and band selection, enhance the clarity and interpretation of volcanic phenomena observed in the imagery.
https://medium.com/@martin2kelko/googleearth-engine-api-process-to-visualize-lava-flow-eruption-at-fernandina-volcano-in-jupyter-591dce18e572

## How to Use
To use this code, follow these steps:

1. **Set Up Your Environment**
   - Ensure you have a Google Earth Engine account.
   - Install the necessary libraries: `earthengine-api` and `IPython`.

2. **Authenticate and Initialize the Earth Engine API**
   ```python
   import ee
   from IPython.display import Image, display

   # Authenticate the Earth Engine API
   ee.Authenticate()

   # Initialize the Earth Engine API
   ee.Initialize()

   # Define coordinates for the polygon
coords = [
    [-91.674385, -0.517037],
    [-91.674385, -0.253371],
    [-91.364708, -0.253371],
    [-91.364708, -0.517037],
    [-91.674385, -0.517037]
]

# Create a polygon geometry
polygon = ee.Geometry.Polygon(coords)

# Define the exact date range (March 06, 2024)
start_date = '2024-03-06'
end_date = '2024-03-07'  # End date is exclusive, so use the next day

# Load the Sentinel-2 ImageCollection for the specified date
sentinel2 = ee.ImageCollection('COPERNICUS/S2_HARMONIZED') \
             .filterDate(start_date, end_date) \
             .filterBounds(polygon) \
             .filterMetadata('CLOUDY_PIXEL_PERCENTAGE', 'less_than', 100)

# Get the first image from the collection (if available)
image = sentinel2.first()

# Load the Sentinel-2 ImageCollection for the specified date
sentinel2 = ee.ImageCollection('COPERNICUS/S2_HARMONIZED') \
             .filterDate(start_date, end_date) \
             .filterBounds(polygon) \
             .filterMetadata('CLOUDY_PIXEL_PERCENTAGE', 'less_than', 100)

# Get the first image from the collection (if available)
image = sentinel2.first()

# Define True-color visualization parameters
natural_color_vis = {
    'bands': ['B4', 'B3', 'B2'],
    'min': 0,
    'max': 3000,
    'gamma': 0.9  # Lower gamma to increase saturation
}

# Define False-color visualization parameters (for the lava flow) with increased contrast
false_color_vis = {
    'bands': ['B12', 'B11', 'B8A'],
    'min': 0,
    'max': 4000,  # Increase max value for higher contrast
    'gamma': 0.8  # Lower gamma to enhance saturation
}

# Function to add thermal radiation (the lava flow) to the true color image
def add_thermal_radiation(natural_img, false_img):
    # Extract the lava flow area from the false color image using a threshold
    lava_mask = false_img.select('vis-red').gt(150)  # Adjust the threshold as needed
    lava_flow = false_img.updateMask(lava_mask)
    
    # Combine the natural image with the lava flow
    combined = natural_img.blend(lava_flow)
    return combined

# Check if there is an image available for the specified date
if image is not None:
    # Apply visualization parameters to the natural and false color images
    natural_color_image = image.visualize(**natural_color_vis)
    false_color_image = image.visualize(**false_color_vis)
    
    # Add the thermal radiation (the lava flow) to the true color image
    combined_image = add_thermal_radiation(natural_color_image, false_color_image)
    
    # Get a URL to visualize the combined image
    url = combined_image.getThumbURL({
        'dimensions': 1024,  # Increase dimensions for better resolution
        'region': polygon.getInfo()  # Ensure the region is set to your polygon
    })
    
    # Display the combined image directly in the notebook with specific width and height
    display(Image(url=url, width=1200, height=1000))  # Adjust width and height as needed
else:
    print("No image available for the specified date and criteria.")
