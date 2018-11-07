
# Frame Machine Notation (FMN) 0.1


FMN Syntax | Definition | Example | Gist
:---: | :---: | :---: | :---:
<span>#</span> | Controller | #LoggedOut |
-block- | Block Section | -machine- |
$ | Frame State | $Begin | 
@ | Frame Event | @ | 
@&#124;message&#124; or &#124;message&#124; | Frame Event Message Selector | &#124;buttonClick&#124;
@&#124;>>&#124; | Frame Start Message | @&#124;>>&#124; | 
@&#124;<<&#124; | Frame Stop Message | @&#124;<<&#124; | 
@&#124;>&#124; | Frame Enter State Message | @&#124;>&#124; |
@&#124;<&#124; | Frame Exit State Message | @&#124;<&#124; |
@[param] or [param] | Frame Event parameter attribute | @[firstName] |
@^ or ^(expr) | Frame Event return attribute | ^("YES!") | 
^ | Return statement | ^ |
->> | Change state operator | ->> $Working | [Change State](https://gist.github.com/frame-lang/ebec407ea6956e1d9dd7d0c3aa6a0df1)
-> | Transition operator | -> $Working | [Transition](https://gist.github.com/frame-lang/47cb1e87715c38861a5b0ebbda5e3bce)


## General Syntax

```
// this is a comment
```

## Controller

A controller is a Frame machine, or automaton, that contains blocks (or sections) that define the internal structure of the machine.  From an implementation standpoint, the reference architecture for implementing a Frame controller is defined as an object-oriented class, but this is for convieniece and not a requirement.

```
// FMN

#MyController

// Pseudocode implementation

class MyController {}
```

## Interface Block

```
// FMN

-interface-                             // interface block declaration
        m1                              // no parameters or return value
        m2[param1 param2]               // typeless parameters
        m3[param1:string param2:int]    // typed parameters
        m4:boolean                      // typed return value
        m5[param4 param5:string]:int    // mixed parameters and return value
        m6 @(|6m|)                      // message alias
        
// Pseudocode implementation

	func m1() { 
		var e = new FrameEvent("m1")
		_state(e) 
	}
  
	func m2(param1 param2) { 
		var params = { "param1":param1, "param2":param2 }
		var e = new FrameEvent("m2", params)
		_state(e) 
	}
  
	func m3(param1:string, param2:int) { 
		var params = { "param1":param1, "param2":param2 }
		var e = new FrameEvent("m3", params)
		_state(e) 
	}

  
	func m4():boolean { 
		var e = new FrameEvent("m2")
		_state(e) 
		return (boolean) e._return
	}

  
	func m5(param4, param5:string):int { 
		var params = { "param1":param1, "param2":param2 }
		var e = new FrameEvent("m2", params)
		_state(e) 
		return (int) e._return
	}

  
	func m6() { 
		var e = new FrameEvent("6m")
		_state(e) 
	}

  
```

## Machine Block

```
// FMN

-machine-                             		// machine block declaration
	$Begin 					// $    -- token indicates state definition
		|>>| 				// |>>| -- start event selector
			-> $Working 		// ->   -- transition token
                        ^			// ^    -- return token
                
        $Working => $Default 			// =>   -- Dispatch operator to send event to $Default
		|>| startWorking() ^		// |>| 	-- enter event selector
		|<| stopWorking() ^		// |<|  -- exit event selector
             
        $End					// $End state has no event handlers. 
                
        $Default				// $Default parent state or "mode"
		|<<| -> $End ^			// |<<|  -- stop event selector token

// Implementation
 
    	// -machine-
    
   	var _state(e:FrameEvent) = Begin	// initialize start state
	
	func Begin(e:FrameEvent) {		// $Begin
		if (e._msg == ">>") {		// |>>|
		    _transition(Working)	// -> $Working
		    return			// ^
		}
	}
    
	func Working(e:FrameEvent) {		// $Working
		if (e._msg == ">") {		// |>|
		    startWorking()		// startWorking()
		    return			// ^
		}
		if (e._msg == "<") {		// |<|
		    stopWorking()		// stopWorking()
		    return			// ^
		}

		Default(e)			// $Working => $Default
	}  

	func End(e:FrameEvent) {		// $End
	}  

	func Default(e:FrameEvent) {		// $Default
		if (e._msg == "<<") {		// |<<|
		    _transition(End)		// -> $End
		    return			// ^
		}	
	}  
	
	func _transition(newState:FrameState) {
		_state(new FrameEvent("<"))	// send exit event
		_state = newState		// change state
		_state(new FrameEvent(">"))	// send enter event
	}


```

## Event Handlers

	$Working => $BaseStateOrMode
		|>>| startMachine() ^		// start machine event
		|>| enterState() ^		// enter state event
		|e1| processE1(@[x] @[y]) ^	// packed param passing
		|e2|[x y] processE2(x y) ^	// unpacked param passing
		|<| exitState() ^		// exit state event
		|<<| stopMachine() ^		// stop machine event
		
		
	func Working(e:FrameEvent) {		// $Working
		if (e._msg == ">>") {		// |>|
		    	startMachine()		// startMachine()
		    	return			// ^
		}
		if (e._msg == ">") {		// |>|
		    	enterState()		// enterState()
		    	return			// ^
		}
		if (e._msg == "e1") {		// |e1|
			processE1( 		// processE1(@[x] @[y])
				e.params['x']
			)	e.params['y']
		    	return			// ^
		}
		if (e._msg == "e2") {		// |e1|
			var x = e.params['x']	// [x y]
			var y = e.params['y']
			processE2(x,y)		// processE2(x y)
		    	return			// ^
		}
		if (e._msg == ">") {		// |>|
		    	enterState()		// enterState()
		    	return			// ^
		}
		if (e._msg == "<") {		// |<|
		    	exitState()		// exitState()
		    	return			// ^
		}
		if (e._msg == "<<") {		// |<<|
		    	stopMachine()		// stopMachine()
		    	return			// ^
		}

		BaseStateOrMode(e)		// $Working => $BaseStateOrMode
	} 
