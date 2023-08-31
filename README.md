# Fastapi patch delete request code along


## Making patch requests  

A patch request is made to the server to partially update a resource. 
We may want to update a specific students details such as their home_town, age or last_name and leave the other fields intact. This can be done by making a patch request targeting the specific resource along with the associated data used to update the resource.       
To create an endpoint for a patch request in fastapi, we decorate the function that will respond to patch requests with the patch method of the app instance we had created earlier.  

i.e     
```
@app.patch('/students/{id}')
def partial_update_student(id : int, payload)
    pass
```    
We have not annotated our payload with a schema as the Schema to be used has not been created. If we try to use the initial `StudentSchema` we had created we will forced to add all the fields when making the patch request which is not supposed to be the case with a patch request.    
Our schema must be flexible to give room to exclude fields we don't want to update. 
let us create the `StudentUpdateSchema`     
```
from typing import Optional

class StudentUpdateSchema(BaseModel):
    id: Optional[int]
    last_name: Optional[str]
    first_name: Optional[str]
    age: Optional[int]
    home_town: Optional[str]

    class config:
        orm_mode= True

```     
The Schema above will set all unset values to None which will allow us to target only set values.       
Update the patch function so that the payload is annotated with the schema above.   
```
@app.patch('/students/{id}')
def partial_update_student(id : int, payload:StudentUpdateSchema)
    pass
```     
We then add the logic for updating the student when the patch request is made. 

