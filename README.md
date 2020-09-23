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
8. Create bone geometry (mesh)
9. Create a handler function that will run on each frame
10. Write a function that iterates through all frames and renders a png of each one 


Okay, let's get started!

## Steps Detailed:

### 1. Import tsv file containing the Qualisys (x, y, z) data for each marker on each frame into Blender.

  ```python
  #At top of code
  import csv
  
  #file path of the tsv data
  input_tsv = r"SteveWalking0004.tsv"
  
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
Then we can iterate through our list of markers and create an empty object at each marker position on the first frame.

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

### 5. Create an armature object 💀

An armature in Blender can be thought of as a skeleton, which can have many bones. We will use this to represent the underlying body performing the action represented by the motion capture data, and eventually we will use this armature to deform a mesh.

  ```python
    #Create armature object
    armature = bpy.data.armatures.new('Armature')
    armature_object = bpy.data.objects.new('Armature', armature)
    #Link armature object to our scene
    bpy.context.collection.objects.link(armature_object)
    #Make armature variable
    armature_data = bpy.data.objects[armature_object.name]
    #Set armature active
    bpy.context.view_layer.objects.active = armature_data
    #Set armature selected
    armature_data.select_set(state=True)
   ```
### 6. Add bones to your armature object 🦴

Create a function that will add a bone to your new armature! We want the bone to connect two markers, since the markers are located at joints so we set one end of the bone (bone head) location to be "empty1" and the other end of the bone (bone tail) location to be "empty2"
  ```python
#adds child bone given corresponding parent and empty
#bone tail will appear at the location of empty
def add_child_bone(bone_name, empty1, empty2):
    #Create a new bone
    new_bone = armature_data.data.edit_bones.new(bone_name)
    #Set bone's size
    new_bone.head = (0,0,0)
    new_bone.tail = (0,0.5,0)
    #Set bone's location to wheel
    new_bone.matrix = empty2.matrix_world
    #set location of bone head
    new_bone.head =  empty1.location
    #set location of bone tail
    new_bone.tail = empty2.location
    return new_bone
  ```
 
Now, based on how you want to arrange the bones, create a list of tuples that save the information of the bone name, and the two markers that it will connect. 
For example: 
  ```python
list_of_bones_order = [('bone0', order_of_markers[1], order_of_markers[3]),
        ('bone1', order_of_markers[3], order_of_markers[0]),
        ('bone2', order_of_markers[3], order_of_markers[2]),
        ('bone3', order_of_markers[19], order_of_markers[18]),
        ('bone4', order_of_markers[18], order_of_markers[4]),
        ('bone5', order_of_markers[4], order_of_markers[7]),
        ('bone6', order_of_markers[7], order_of_markers[8]),
        ('bone7', order_of_markers[18], order_of_markers[11]),
        ('bone8', order_of_markers[11], order_of_markers[14]),
        ('bone9', order_of_markers[14], order_of_markers[15]),
        ('bone10', order_of_markers[22], order_of_markers[26]),
        ('bone11', order_of_markers[26], order_of_markers[27]),
        ('bone12', order_of_markers[27], order_of_markers[28]),
        ('bone13', order_of_markers[28], order_of_markers[29]),
        ('bone14', order_of_markers[29], order_of_markers[31]),
        ('bone15', order_of_markers[29], order_of_markers[32]),
        ('bone16', order_of_markers[29], order_of_markers[33]),
        ('bone17', order_of_markers[25], order_of_markers[34]),
        ('bone18', order_of_markers[34], order_of_markers[35]),
        ('bone19', order_of_markers[35], order_of_markers[36]),
        ('bone20', order_of_markers[36], order_of_markers[37]),
        ('bone21', order_of_markers[37], order_of_markers[41]),
        ('bone22', order_of_markers[37], order_of_markers[39]), 
        ('bone23', order_of_markers[12], order_of_markers[13]),
        ('bone24', order_of_markers[13], order_of_markers[14]),
        ('bone25', order_of_markers[15], order_of_markers[17]),
        #('bone26', order_of_markers[37], order_of_markers[40]),
        ('bone27', order_of_markers[19], order_of_markers[5]),
        ('bone28', order_of_markers[5], order_of_markers[6]),
        ('bone29', order_of_markers[6], order_of_markers[7]),
        ('bone30', order_of_markers[8], order_of_markers[9]),
        ('bone31', order_of_markers[19], order_of_markers[18]),
        ('bone32', order_of_markers[18], order_of_markers[20]),
        ('bone33', order_of_markers[18], order_of_markers[21]),
        ('bone34', order_of_markers[20], order_of_markers[23]),
        ('bone35', order_of_markers[21], order_of_markers[24]),
        ('bone36', order_of_markers[23], order_of_markers[22]),
        ('bone37', order_of_markers[24], order_of_markers[25]),
        ('bone38', order_of_markers[22], order_of_markers[25]),
        ('bone39', order_of_markers[3], order_of_markers[18]),
        ('bone40', order_of_markers[29], order_of_markers[30]),
        ('bone41', order_of_markers[37], order_of_markers[38]),
        ('bone42', order_of_markers[18], order_of_markers[22]),
        ('bone43', order_of_markers[18], order_of_markers[25]),
        ('bone44', order_of_markers[23], order_of_markers[27]),
        ('bone45', order_of_markers[24], order_of_markers[35]),
        ('bone46', order_of_markers[12], order_of_markers[19]),
        ('bone47', order_of_markers[27], order_of_markers[30]),
        ('bone48', order_of_markers[35], order_of_markers[38]),
        ('bone49', order_of_markers[8], order_of_markers[10])]
        
        
#helper to create armature from list of tuples
def tuple_to_armature(bones):
    for bone_name, bone_head, bone_tail in bones:
        add_child_bone(bone_name, bone_head, bone_tail)
        
#create all bones for skeleton body and hands
tuple_to_armature(list_of_bones_order)

  ```

... Now you have your animation! 🎦

Parts not covered in this tutorial: 
- Animating a rigid body like the cyr wheel
- Exporting XML data
- Creation of virtual markers


