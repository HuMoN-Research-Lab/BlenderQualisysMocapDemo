# BlenderQualisysMocapDemo

Welcome to the demo of importing tsv Qualisys Motion Capture data into Blender and creating a full body animation with it! 👾

## Steps:

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


... Now you have your animation! 🎦

Parts not covered in this tutorial: 
- Animating a rigid body like the cyr wheel
- Exporting XML data
- Creation of virtual markers


