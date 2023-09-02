# Fastapi patch delete request code along


## Making patch requests  

A patch request is made to the server to partially update a resource. 
We may want to update a specific students details such as their home_town, age or last_name and leave the other fields intact. This can be done by making a patch request targeting the specific resource along with the associated data used to update the student.       
To create an endpoint for a patch request in fastapi, we decorate the function that will respond to patch requests with the patch method of the app instance we had created earlier.  

i.e     
```
@app.patch('/students/{id}')
def partial_update_student(id : int, payload)
    pass
```    
We have not annotated our payload with a schema as the Schema to be used has not been created. If we try to use the initial `StudentSchema` we had created we will be forced to add all the fields when making the patch request which is not supposed to be the case with a patch request.    
Our schema must be flexible to give room to exclude fields we don't want to update. 
let us create the `StudentUpdateSchema`     
```
from typing import Optional

class StudentUpdateSchema(BaseModel):
    id: Optional[int] = None
    last_name: Optional[str] = None
    first_name: Optional[str] = None
    age: Optional[int] = None
    home_town: Optional[str] = None

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
```
@app.patch('/students/{id}',response_model=StudentSchema)
def partial_update_student(id:int, payload:StudentUpdateSchema):
    # query student based on the supplied id
    student = session.query(Student).filter_by(id=id).first()

    # raise a not found exception if no match is found
    if not student:
        raise HTTPException(status_code=404, detail="Student not Found")
    
    # iterate over set values and use to update Student instance retrieved from the db
    for key,value in payload.dict(exclude_unset=True).items():
        setattr(student, key, value)

    # persist the changes in the database
    session.commit()

    # give a response back to the user 
    return student
```  
Let's explain a few lines in the code above.  

`for key,value in payload.dict(exclude_unset=True).items():`  

The payload being an instance of our pydantic model has the dict function which converts the values to a dictionary. Adding `exclude_unset=True` ensures that the payload should only convert values that are not None from our payload.The items() property allows us to unpack the key,value pairs from the dictionary.   

`setattr(student, key, value)`  
We use the key, value pairs from the payload to update the student instance we had retrieved from the database.     
`session.commit()`  
We persist the changes to the database.     
Our patch request is now complete and we can test it using postman.  
![Making Patch Requests](./making%20patch%20request.png)     
Don't forget to set the necessary headers   
![Adding headers to request](./add%20headers%20in%20patch%20requests.png)   

Once you make a patch request you should get a response back in a format similar to the one below.  
![Response from patch request](./Patch%20response.png)      

## Delete Request   

A delete request is made to delete a resource from the server. We receive an id of the resource we query it from the database then delete it.   
To achieve this in fastapi, we will decorate the function with the delete method from of our app instance and add the necessary logic within the decorated function. you r function should receive as a query parameter the id of the student to delete.    

```
@app.delete('/students/{id}')
def delete_student(id: int):
    # add logic for deleting a student from the database 
    pass    
```     
The logic for deleting a student from the database involves querying the student, from the database based on the id passed. Adding conditional branching for unsuccessful queries, deleting the student if found and finally giving a response back to the user.    
```
@app.delete('/students/{id}')
def delete_student(id: int):
    student = session.query(Student).filter_by(id=id).first()
    if not student:
        raise HTTPException(status_code=404, detail="Student not found ")
    session.delete(student)
    session.commit()
    return {"detail":f'Student with id {id} has been deleted successfully'}    
```     

Next we use postman to make a delete request to the endpoint. 
![MAKING A DELETE REQUEST](./delete%20request.png)      
We should get a response back as below.     
![SUCCESSFUL DELETE REQUEST](./successful%20delete%20request.png)      

This brings us to the end of making patch and delete requests in fastapi. 


Practice Makes perfect and therefore do a lot of practice in order to enforce whatever you have learnt in this short series guide. 

In the next section we shall discuss relationships in sqlalchemy and how to render them in fastapi endpoints.   

  









