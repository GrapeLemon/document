```
xxx(){
	Future<Response> future = xxxService.search(XXRequext);
	Response resp = future.get();
	resp.getxxx();
	doSomething();    
}

xxx(){
    xxxService.search(XXRequext, XXXCallback);
}

public class XXCallback<Response> implemst Callback{

	vodi call(Respono resp){
        
        resp.getXX()
		doSomething();  
	}

}


```

