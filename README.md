# Tech_challenge
 **We have a nested object. We would like a function where you pass in the object and a key and 
get back the value**. 

Example Inputs 

object = {“a”:{“b”:{“c”:”d”}}} 

key = a/b/c 

object = {“x”:{“y”:{“z”:”a”}}} 

key = x/y/z 

value = a 

--
def get_value_from_nested_object(obj, key): 

    keys = key.split('/') 
    
    result = obj 
    
    for k in keys: 
    
        if k in result: 
        
            result = result[k] 
            
        else: 
        
            return None  # Key not found in the  object 
            
    return result 
    
    

-----------------------------------------------------------

----------------------------------------------------

Example Input for the above question 



object1 = {"a": {"b": {"c": "d"}}} 

key1 = "a/b/c" 

value1 = get_value_from_nested_object(object1, key1) 

print(value1)   


object2 = {"x": {"y": {"z": "a"}}} 

key2 = "x/y/z" 

value2 = get_value_from_nested_object(object2, key2) 

print(value2) 



Final output for the above question will be 


d  

a 

Acoording to my obersvation this is an accurate code for the question,the get_value_from_nested_object function takes the object and key as input and iterates through the nested structure using the provided key. If the key exists at each level, it continues to the next level until it reaches the desired value. If the key is not found at any level, it returns None.


