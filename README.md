# HellRacer-Workshop

Hello HellRacer is a race game where players race against their own ghost which represents their best time. 
We made the game in unreal engine with C++. I had watched some videos for learning C++ but i cant say that i had enough knowledge for participate in making a race game. It was mostly the knowledge i had before and ChatGbt.

My responsibility was the car movement system. First I had no idea where to start then with some youtube videos made a simple movement class based on Character class. But it was based on Unreal´s character movement component, I wanted to make a custom movement system from scratch. With some research on internet I started to create a new movement based on pawn. However, creating from pawn have some disadvantages, there were no physic. Of course with some work I could make a simple custom physic system. But it was too stressfull then and I wasn´t so sure about if I could do it before we run out of time so I went back to character movement. 

We decided as programmer to make a main class for the car and attach all other scripts to that, movement, drift, particle and other stuff. All my moving logic was based on character movement component values. But there was a big problem. Whenever you rotated the car during the deacceleration, it started to slide away like if it was on ice. I gave 1 week to find a solution but couldn´t even find the problem itself. Instead I cheated a bit. By making a custom logic for velocity that forces it to interpolate to a lower value I got the effect I wanted. 


Here is how i made the acceleration logic. 
<details>
  <summary>Click to expand</summary>
  
```csharp
void UCarMovementComponent::AccelerateMovement(float InputValue, bool bIsAccelerating)
{

      // this is the variable I use later in main class to set the input movement
	CurrentAccelerationForInput = FMath::Lerp(CurrentAccelerationForInput, 1, AccelerateUpSpeed);
      // I use this bool to prevent rotate smoothly when driving reverse.
	isRotatingSmooth = true;

	CharacterMovementComponent->MaxAcceleration = SetMaxAcceleration;
	CharacterMovementComponent->BrakingFrictionFactor = BrakeFriction;
	CharacterMovementComponent->MaxWalkSpeed = CurrentTopSpeed;

	if (AccelerationFromCharacter > SetMaxAcceleration)
	{
		AccelerationFromCharacter = SetMaxAcceleration;
	}

	if (CurrentSpeed < MinimumSpeedToRotateInNormalSpeed && InputValue == 1)
	{
		WorldRotateAlpha += 0.05 * GetWorld()->GetDeltaSeconds();
	}
	else if (CurrentSpeed > MinimumSpeedToRotateInNormalSpeed && InputValue == 1)
	{
		WorldRotateAlpha += 0.01 * GetWorld()->GetDeltaSeconds();
	}

	if (WorldRotateAlpha > 1)
	{
		WorldRotateAlpha = 1;
	}


	if (bIsAccelerating)
	{
		AccelerationFromCharacter += InputValue * AccelerationSpeed * GetWorld()->GetDeltaSeconds();
	}
	else
	{
		AccelerationFromCharacter -= InputValue * AccelerationSpeed * GetWorld()->GetDeltaSeconds();
	}

        CharacterMovementComponent->MaxAcceleration = AccelerationFromCharacter;
}
```
</details>
