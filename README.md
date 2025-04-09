# Proposal for the Polyshell Challenge
Proposal for the Polyshell Challenge

## Proposed Solution: Objectives & Goals
In my proposed solution, I will utilize a combination of well-established geometry manipulation techniques to effectively process and analyze spatial data. These techniques include:

1. **Convex Hull** 
2. **Buffering**
3. **Geometry Intersection and Area Calculation**
4. **Transformation between Polygon and Raster**
5. **Boundary Refinement** – Capturing the general shape of the boundary by identifying local minima along the linestring when necessary.

Rather than aiming for a **single "best" solution**, my proposed algorithm aim to provide a **flexible framework** (see attached flowchart) that guides users in effectively combining and utilizing these techniques based on their specific needs and requirements.

The attached example demonstrates that even with a relatively small amount of code, a combination of these techniques can already produce highly effective results. This approach brings several key advantages:

### **1. Simplicity and Transferability**

The selected geometry manipulation techniques are all well-established and predominantly based on **GDAL**, making them both reliable and easy to implement. With GDAL wrappers dedicated for various programming languages (e.g., **PostGIS** in PostgreSQL, **gdal-async** in Node.js), the methodology can be effortlessly adapted across different platforms and environments.

### **2. Leveraging State-of-the-Art Libraries for Performance Optimization**

GDAL is widely recognized as a **state-of-the-art** library for geospatial data processing. By utilizing GDAL's highly optimized algorithms, the proposed solution ensures efficient performance while handling large shapefiles.

### **3. Customizability Through Parameter Selection**

One of the key strengths of this approach is its flexibility. Users can fine-tune their results by selecting different **combinations of techniques** and adjusting **parameters** to best match their requirements. Whether prioritizing accuracy, computational efficiency, or boundary generalization, users can tailor the process to achieve an optimal balance between precision and processing time.

As an open-source solution, I hope this framework will be valuable to users across different programming languages. By providing a flexible and adaptable approach, it can serve as a foundation for further development and customization to meet diverse needs.

## Methodology: Tools, Services, and Data
### **1 - Creating the Convex Hull**

Although the goal is to preserve significant concavities, the **Convex Hull** serves as a useful starting point for polygon simplification. It captures key vertices that can act as reference points for rotation during boundary simplification (Step 4A). Additionally, the Convex Hull can be used to **truncate excessive areas** generated from **buffering** (Step 2) and **raster upscaling** (Steps 3B/4B).

### **2 - Adding a Buffer to the Polygon**

Applying a **buffer** (yellow area in Fig. 1) is particularly useful when simplifying a **single closed linestring** (i.e., the exterior boundary of a polygon). The buffer **expands the boundary outward**, ensuring that the entire area of the original polygon is retained. For highly detailed polygons, such as country boundaries from OpenStreetMap, adding a buffer alone can significantly reduce the number of vertices (see Table 1).

![fig1](https://github.com/yinyingip/polyshell-proposal/blob/main/fig1.png)
<sub>Fig. 1. Example of Polygon Simplification Algorithm using the country boundary of Germany from OpenStreetMap. Grey: The original Polygon; Yellow: adding Buffer; Blue: Difference between Convex Hull and the Grey area; Orange: Area to be appended to the original polygon; Red Dots: Convex Hull vertices</sub>

### **3A - Identifying Concavities Using Polygon Difference**

Concavities can be detected by computing the **difference** between the **Convex Hull** (optionally with an added buffer) and the original polygon. This results in multiple **small polygons** (blue areas in Fig. 1), each with two boundary components:

- A densely delineated linestring from the **original polygon** (**target_line**).
- A simplified segment from the **Convex Hull**, defined by two vertices (**ref_line**).

The area of these small polygons, or their proportion relative to the original polygon's area, directly correlates with the significance of the concavity.

- **If the area is small (below a defined threshold)** → The concavity is minor and should be **smoothed out**. In this case, the **target_line** in the original polygon is replaced by the **ref_line** (orange areas in Fig. 1 indicate the areas appended to the original polygon).
- **If the area is large (above the threshold)** → The concavity is significant, and the **general shape of the target_line should be preserved**. However, if the target_line has an excessively high number of vertices, it will be further simplified in Step 4A.

### **4A - Simplifying the Linestring Using Local Minima**

Before processing the **target_line**, the polygon and its two boundary components (**target_line** and **ref_line**) are rotated so that **ref_line aligns with the x-axis** (Fig. 2a). The **local minima** along the target_line provide an approximation of its shape but with fewer vertices, resulting in a simplified yet representative boundary (Fig. 2a).

### **3B/4B - Polygon-Raster Transformation**

An alternative approach to approximating the polygon boundary with fewer vertices is through **polygon-to-raster transformation**. This involves:

1. **Converting the polygon** (ideally with an added buffer from Step 2) into a **low-resolution raster** (e.g., 100 × 100 pixels).
2. **Re-polygonizing** the rasterized polygon, generating a simplified shape based on pixelized boundaries (Fig. 3).