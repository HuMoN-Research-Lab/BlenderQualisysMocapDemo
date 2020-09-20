# BlenderQualisysMocapDemo

Welcome to the demo of importing tsv Qualisys Motion Capture data into Blender and creating a full body animation with it! 👾

Here's a general overview of the steps that are taken to produce the animation:
## Steps Overview:
1. Import tsv file containing the Qualisys (x, y, z) data for each marker on each frame into Blender.
2. Create a function that will take the list of all data in "file" and return just the (x, y, z) data for each marker on one frame (frame # is given as a parameter to the function)
3. Create an array of marker names 
4. Add an empty object at each marker location for one frame
5. Create an armature object
6. Add bones to your armature object
7. Create a parent relationship between the head of a bone and an empty, and the tail of the same bone with a different empty
8. Add bones to your armature object
9. Create bone geometry (mesh)
10. Create a handler function that will run on each frame
11. Write a function that iterates through all frames and renders a png of each one 


Okay, let's get started!

## Steps Detailed:

### 1. Import tsv file containing the Qualisys (x, y, z) data for each marker on each frame into Blender.

  ```python
  #At top of code
  import csv
  
  #open file and read marker animation data, convert data into a huge list
  with open(input_tsv, "r") as tsv_file:
   file = list(csv.reader(tsv_file, delimiter='\t'))
  ```
  Now the variable "file" holds a list that contains 
  
### 2. Create a function that will take the list of all data in "file" and return just the (x, y, z) data for each marker on one frame (frame # is given as a parameter to the function)

This can be done in many ways, here's my approach: 

This function parses the data into an array of arrays, where the outer array holds every [x, y, z] array that represents a marker location. 

We have to find the row and column of where the data actually starts in the tsv. We know that in the Qualisys data, the header ends at row 11, and the left_header ends at column 2. 

```python
  header_end = 11
  left_header_end = 2

  # Create 2D array "arr" to hold all 3D coordinate info of markers
  #return an array containing all marker locations at given frame
  def create_data_arr(frame):
      current_row = file[frame + header_end]
      cols, rows = (3, int((len(current_row) - 2) / 3))
      arr = [[None]*cols for _ in range(rows)]
      count = 0
      count_row = 0
      for x in range(left_header_end, len(current_row)):
          arr[count_row][count] = current_row[x]
          count += 1
          if (count == 3):
              count = 0
              count_row += 1
      return arr; 
``` 
      
Then, we can extract the data from just the first frame as an array
```python
   arr = create_data_arr(frame)
``` 
   
### 3. Create an array of marker names 🧩

For this kind of Qualisys data, we know that the marker names are contained in the 9th row of the data.

So, we create a list of the marker names in order so that we can name each of our marker objects late on. 
   ```python
    marker_names_row = 9
    
    #Create an array of marker names 
    current_row = file[marker_names_row] 
    name_arr = []
    for index in range(1, len(current_row)):
        name_arr.append(current_row[index])
   ```

### 4. Add an empty object at each marker location for one frame
At the top of the code, import bpy which we will need to create a Blender object
   ```python
   #At the top of the code
   import bpy
   ```
Then we can iterate through our list opf markers and create an empty object at each marker position on the first frame.

   ```python
    #Create empties at marker positions
    #position in the name array
    name = 0
    #an array to hold all marker objects
    order_of_markers = []
    #iterate through arr and create an empty object at that location for each element
    for col in arr:
    # parse string float value into floats, create Vector, set empty position to Vector
    # multiply by .001 to scale down data
    coord = Vector((float(col[0]) * 0.001, float(col[1]) * 0.001, float(col[2]) * 0.001))
    #Add an empty object to the scene
    bpy.ops.object.add(type='EMPTY', location=coord)  
    #assign the empty to variable 
    mt = bpy.context.active_object  
    #get name from name array "name_arr"
    mt.name = name_arr[name]
    #increment name iter of empty names array 
    name += 1
    #link empty to this scene
    bpy.context.scene.collection.objects.link( mt )
    #set empty location
    mt.location = coord
    #set empty display size
    mt.empty_display_size = 0.1
    #add empty to array order_of_markers so we can later access it 
    order_of_markers.append(mt)
   ```

<p align="center">
 <img src="https://user-images.githubusercontent.com/44556715/93720789-41092180-fb59-11ea-8d46-7e0d07fa12c0.png">
</p>

### 5. Create an armature object


... Now you have your animation! 🎦

Parts not covered in this tutorial: 
- Animating a rigid body like the cyr wheel
- Exporting XML data
- Creation of virtual markers


