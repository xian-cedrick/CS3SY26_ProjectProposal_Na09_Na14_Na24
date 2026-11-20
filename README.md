# CS3SY26_ProjectProposal by Aline Teresa Mendoza from CS Unit of PSHS


## Project Proposal Template:

### Project Title or Name of the Application	: Physics Force Calculator
### Section			   	      : Sodium
### Group Members			    : Dela Cruz, Gardose, Sinno
### Date (Proposal Date)  : Nov 13, 2026

### Project Description:
A brief summary of the project description and the need for it:
  - The Physics Force Calculator (PhyFC) aims to aid students in solving multi-step problems related to calculating net forces, x- and y-components of forces, static and kinetic friction, and other common mechanical forces. It is needed because students often make mistakes with calculations, sign conventions, and formulas used. This application intends to have a working and clean user interface (UI) for users to choose what they need help solving with—they will be able to choose between different tabs for force calculations, and simple guiding steps. However, this project would not be able to solve questions entirely on its own and acts only as a tool to its users. 

**Description of system components, features, and expected outputs:**

_Class 1: Force (ABC): ⇐ Abstract Base Class_
The blueprint for any single force acting on an object which uses the abc module to import ABC and abstractmethod.
Attributes:
- _label: str (name or type of force)
- _axis: str (coordinate axis of force, the x- or y-axis, aligned with the type of surface: horizontal/vertical on a flat plane, respectively; parallel/perpendicular on an inclined 			plane, respectively.)
- _sign: int (direction sign along the axis which a force is on: for flat and inclined planes, on the x-axis, +1 is to the right/up the inclined plane, -1 is to the left/down the 			inclined plane; on the y-axis, +1 if directed up/away from the inclined plane, -1 if directed down/into the inclined plane.)
	
Methods:
- calculate_magnitude() -> float @abstractmethod 
	- forces each concrete subclass to define its formula for its magnitude.
- get_signed_component() -> float (getter method)
	- returns self._sign * self.calculate_magnitude() so directional sign is applied and addresses mistakes in sign-convention
		
Interactions:
- Inherited by force subclasses and each of them overriding calculate_magnitude()
- Used by another class, PhySystem, which stores Force objects and reads each object’s axis and get_signed_component() to calculate net forces.
  
Feature: Gives every force the same structure, so each one has a label, an axis, and a direction sign.

Expected output: The signed value of a force (positive or negative) through get_signed_component().

_Class 2: PhySystem:_
- This class represents the physical setup—it holds the object’s mass, the surface’s inclined angle, and the list of the forces acting on the object. It will compute the net forces as well as the acceleration.

Attributes:
- _mass: float (mass of the object in kilograms, kg).
- _inclined_angle: float (surface angle in degrees—0.0 for flat planes, and greater than 0.0 for inclined planes)
- _forces: list[Force] (this stores the forces in the system)
	
Methods: 
- add_force(force: Force) -> None 
	- adds a Force object to _forces.
- calculate_fnet_x() -> float 
	- computes the net force along the x-axis (horizontal on a flat plane; parallel to the inclined surface when _inclined_angle > 0.0)
- calculate_fnet_y() -> float
	- computes the net force along the y-axis (vertical on a flat plane; perpendicular to the inclined surface when _inclined_angle > 0.0)
- calculate_acceleration() -> float
	- computes acceleration along the x-axis using the result of calculate_fnet_x() and _mass.
	- uses the formula a = fnet_x / _mass
		
Interactions:
- Composes or manages Force objects by accepting Force objects via add_force() and separates their "x" and "y" signed components using _inclined_angle in order to interpret the axes (horizontal or vertical | parallel or perpendicular) and compute net forces and acceleration.
- Connects to the intended user interface by receiving _mass, _inclined_angle, and force inputs, then outputs net forces and acceleration.
- It groups the forces by their axis (x vs. y): When calculating net force along the x-axis (calculate_fnet_x()) or y-axis (calculate_fnet_y()), PhySystem looks at each Force object's axis and pulls its signed component (get_signed_component()).
  
Feature: Collects all forces acting on the object and combines them, for flat or inclined surfaces.

Expected output: Net force along the x-axis (N), net force along the y-axis (N), and acceleration (m/s²).

_Class 3: AppliedForce(Force):_
- It stores how strong the force is, its angle, and which axis the angle is measured from (_reference_axis), and the stuff inherited from abstract base class, Force.
  
Attributes: 
- _magnitude: float 
- _angle: float
- _reference_axis: str

Methods:
- calculate_magnitude() -> float
	- returns self._magnitude
- calculate_components() -> tuple[float, float] (the tuple contains the x- and y-components, respectively.
	- resolves an angled applied force into its x and y-components using trigonometry.
	- If _reference_axis is from the horizontal: Fx=F cos  ; and, Fy=F sin 
	- If _reference_axis is from the vertical: Fx=F sin  ; and, Fy=F cos 
		
Interactions:
- Inherits from Force and uses the method get_signed_component() so direction signs are consistently applied
- Produces AppliedForce components via calculate_components() for angled forces so each component (x and y) can be registered in PhySystem through add_force().

Feature: Lets the user enter a push or pull, with an angle measured from the horizontal or vertical.

Expected output: The force’s x- and y-components (N).

_Class 4: Weight(Force)_
- It stores the object's mass (_mass), gravity (_gravity, defaulting to 9.81), and the ramp angle (_inclined_angle), and the stuff inherited from abstract base class, Force.
  
Attributes: 
- _mass: float 
- _gravity: float = 9.81
- _inclined_angle: float

Methods:
	calculate_magnitude() -> float
		- computes the weight magnitude along _axis
		- W = self._mass * self._gravity (along the y-axis on a flat surface)
		- W_parallel = self._mass * self._gravity sin  (along the x-axis on an inclined plane)
		- W_perpendicular = self._mass * self._gravity cos  (along the y-axis on an inclined plane)
		- The components of weight on inclined planes both act in the negative direction.

Interactions:
- Inherits from Force and uses get_signed_component() so downward or down the inclined direction signs are consistently applied (_sign = -1).
- Registered in PhySystem via add_force() so its magnitude on a flat plane is included or both its x- and y-components participate in calculate_fnet_x(), calculate_fnet_y(), and calculate_acceleration().

Feature: Computes the weight of the object using its mass and gravity (9.81 m/s² by default), including inclined surfaces.

Expected output: Weight in N, or its parallel and perpendicular components on an incline, always pointing down (-).

_Class 5: NormalForce(Force)_
- It stores just how strong the force is(_magnitude), and the stuff inherited from abstract base class, Force.
  
Attributes:
- _magnitude: float

Methods:
- calculate_magnitude() -> float
	- returns self._magnitude 
		
Interactions:
- Inherits from Force and uses get_signed_component() so its perpendicular direction on inclined planes as well as on flat surfaces is consistently applied.
- Gets its magnitude from the user’s input.
- Registered in PhySystem net force and acceleration calculations once add_force() is used.
  
Feature: Lets the user enter the normal force from the surface.

Expected output: The normal force in N, pointing away from the surface (+).

_Class 6: FrictionForce(Force)_

Attributes:
- _coefficient: float 
- _normal_force: float
	
Methods:
- calculate_magnitude() -> float
	- overriden by subclasses
		
Interactions:
- Inherits from Force and uses get_signed_component() so its direction is consistently applied.
- Gets its coefficient and normal force from the user’s input.
  
Feature: Holds the coefficient and normal force that both types of friction share.
Expected output: None.

_Class 7: StaticFriction(FrictionForce)_
	
Attributes: (inherited from FrictionForce)

Methods:
-	calculate_magnitude() -> float
	- computes maximum static friction using fs,=sN, or equals the applied force up to that maximum.
		
Interactions:
- Inherits from FrictionForce.
- Gets its coefficient and normal force from the user’s input.
- Registered in PhySystem via add_forces().

Feature: Computes friction for an object that is not yet moving.

Expected output: Static friction in N, up to the maximum value (μs × N).

_Class 8: KineticFriction(FrictionForce)_

Attributes: (inherited from FrictionForce)

Methods:
-	calculate_magnitude() -> float
	- computes kinetic friction using fk=kN, with its direction opposing the object’s motion.
		
Interactions:
- Inherits from FrictionForce.
- Gets its coefficient and normal force from the user’s input.
- Registered in PhySystem via add_forces().

Feature: Computes friction for an object that is already moving.

Expected output: Kinetic friction in N (μk × N), opposite to the motion.

User Interface (UI)
- Gets inputs from the user by asking for basic info such as whether the surface is flat or inclined, and if it is inclined, what the angle is, as well as, the object’s mass, and which forces are acting on the object.
- This data may now be used to register the chosen forces and setup details to the PhySystem and the Force subclasses—these will calculate the forces along the x and y-axis and compute for the acceleration.
- Results should also be displayed clearly by showing the calculated_magnitudes, net forces, acceleration, or vector compositions on the output—making the results easy to read and understand. Units will also be included.
- Tracks history by keeping a list of all calculations for the current session so that the user can review them, and will reset once a new run is initiated.

Description of your target demographics as the user of your application:
 
- Main target users are high school students (Grades 9-12) taking introductory physics. Because students often make mistakes with calculations, formulas, and direction signs when solving force problems with multiple steps, they need a tool that guides them without solving the problem for them. 



 <img width="825" height="638" alt="projectproposal drawio" src="https://github.com/user-attachments/assets/8c5825f2-a41e-40ac-9bc0-7612ddc6da3c" />


### Goals:

In the long term, we hope that PhyFC will help high school students be more confident in their problem-solving involving forces. With the guiding steps, they can make fewer mistakes. It can also become a useful study tool and be improved over time with more forces and features.

**Objective 1:** Code the Python classes for each subclass of forces (Applied Force, Weight, Normal Force, Static Friction Force, and Kinetic Friction Force) using inheritance from the abstract base class, Force.

**Objective 2:** Code the PhySystem class that stores the forces in the system, ensures correct direction sign to each force in the system, and accurately calculates the net force and acceleration.

**Objective 3:** To build an easy-to-use interface with separate tabs for each type of force and their corresponding calculations.

**Objective 4:** To show the formula, values, and result for each calculation, as well as, to display clear error messages for invalid inputs, such as a mass of zero or less and other common errors.


[Click here for Rubrics and Sample projects](https://docs.google.com/document/d/1xKd1CqK6DcNWS2_oJPBNtY5n3I3Hqiw8FY3W295Iqfk/edit?usp=sharing)
