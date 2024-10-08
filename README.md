itch.io: https://yrgo-game-creator.itch.io/hellracer

# HellRacer-Workshop
![image](https://github.com/user-attachments/assets/5107e2eb-72b2-4d48-b820-dae4a683f4fa)
![image](https://github.com/user-attachments/assets/0bdfd15f-4b56-423c-9e26-44cf783dde0b)


Hello HellRacer is a race game where players race against their own ghost which represents their best time. 
We made the game in unreal engine with C++. I had watched some videos for learning C++ but I cant say that I had enough knowledge for participate in making a race game. It was mostly the knowledge I had before and ChatGbt.



My responsibility was the car movement system. First I had no idea where to start then with some youtube videos made a simple movement class based on Character class. But it was based on Unreal´s character movement component, I wanted to make a custom movement system from scratch. With some research on internet I started to create a new movement based on pawn. However, creating from pawn have some disadvantages, there were no physic. Of course with some work I could make a simple custom physic system. But it was too stressfull then and I wasn´t so sure about if I could do it before we run out of time so I went back to character movement. 

# Car Movement

We decided as programmer to make a main class for the car and attach all other scripts to that, movement, drift, particle and other stuff. All my moving logic was based on character movement component values. But there was a big problem. Whenever you rotated the car during the deacceleration, it started to slide away like if it was on ice. I gave 1 week to find a solution but couldn´t even find the problem itself. Instead I cheated a bit. By making a custom logic for velocity that forces it to interpolate to a lower value I got the effect I wanted. 


Here you can see the how the car slides away.

![360961077-98a1d669-53b8-40a6-ba4c-83dad8001524-ezgif com-resize](https://github.com/user-attachments/assets/d064b83a-e05b-4b44-a81c-a9b9fbcb18d0)



Here is how I made the acceleration logic.

![ezgif-1-3e3e1da10e](https://github.com/user-attachments/assets/690f94ba-5148-409c-a2b9-4c8dd5212e1d)



  
```csharp
void UCarMovementComponent::AccelerateMovement(float InputValue, bool bCanApplyAcceleration)
{

      // this is the variable I use later in main class to set the input movement
	CurrentAccelerationForInput = FMath::Lerp(CurrentAccelerationForInput, 1, AccelerateUpSpeed);

	isRotatingSmooth = true;

	CharacterMovementComponent->MaxAcceleration = SetMaxAcceleration;
	CharacterMovementComponent->BrakingFrictionFactor = BrakeFriction;
	CharacterMovementComponent->MaxWalkSpeed = CurrentTopSpeed;

	if (AccelerationFromCharacter > SetMaxAcceleration)
	{
		AccelerationFromCharacter = SetMaxAcceleration;
	}

	if (bCanApplyAcceleration)
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


Here is how I made the rotation logic.


![ezgif-1-a22238efe7](https://github.com/user-attachments/assets/f4d888f5-dcfc-4031-897a-508e9a20feb2)


  
```csharp

void UCarMovementComponent::RotateMovement(float InputValue, AActor* WorldMesh)
{
	float DefaultWorldRotateSpeed = 100;

	bIsRotating = true;
	FVector CurrentVelocity = CharacterMovementComponent->Velocity;
	CharacterMovementComponent->MaxAcceleration = SetMaxAcceleration;


	MyActorRotation = WorldMesh->GetActorRotation();
	float MyWorldRotationYaw = MyActorRotation.Yaw + InputValue * WorldRotateSpeed * GetWorld()->GetDeltaSeconds();
	MyActorRotation.Yaw = MyWorldRotationYaw;
	WorldMesh->SetActorRotation(FRotator(0, MyWorldRotationYaw, 0));


	if (!bIsAccelerating && bIsRotatingSmooth)
	{
		FVector Deceleration = CurrentVelocity.GetSafeNormal() * deAccelerationSpeedDuringRotation * GetWorld()->GetDeltaSeconds();
		FVector NewVelocity = CurrentVelocity - Deceleration;
		CharacterMovementComponent->Velocity = NewVelocity;
	}
	if (CurrentSpeed <= 0) { return; }

	if (bIsRotatingSmooth && !HasLaunched)
	{
		if (CurrentVelocity.Length() <= MinimumSpeedToRotateInNormalSpeed && bIsDrifting == false)
		{
			WorldRotateSpeed = FMath::Lerp(WorldRotateSpeed, SetWorldRotationLowSpeed, WorldRotateAlpha);
		}

		if (CurrentVelocity.Length() > MinimumSpeedToRotateInNormalSpeed && bIsDrifting == false)
		{
			WorldRotateSpeed = FMath::Lerp(WorldRotateSpeed, SetWorldRotationHighSpeed, WorldRotateAlpha);
		}
	}
	else
	{
		WorldRotateSpeed = DefaultWorldRotateSpeed;
	}
}
```


# Animations

I love to making systems for animations so I am happy that I got that part of the project. We had some small lessons how to make animation system in Unreal but now it was something bigger. With some research and collabration with graphic artists from our group I made a system for the animations. 

For the wheels I used the transform nodes inside animation blueprint. As you can see I manipulate the rotations of the wheels with them. 

![image](https://github.com/user-attachments/assets/ede39612-c085-4694-a1e2-1f57fac1ed9f)


  
```csharp
void ACharacterInput::SpinWheel()
{
	if (bIsBrakingHard && !CarIsMovingBackWard)
	{
		return;
	}

	float SpeedReductionFactor = 10.0f;
	float BaseSpinSpeed = 50.0f;
	float AlphaValue = NormalizeMaxSpeed(MovementComponent->CurrentSpeed, MovementComponent->CurrentTopSpeed);

	if (CarIsMovingBackWard)
	{
		BaseSpinSpeed /= SpeedReductionFactor;  // Corrected to adjust BaseSpinSpeed itself
		AnimInstance->WheelSpeed += BaseSpinSpeed * AlphaValue;
	}
	else
	{
		AnimInstance->WheelSpeed -= BaseSpinSpeed * AlphaValue;
	}

	AnimInstance->WheelSpeed = FMath::Fmod(AnimInstance->WheelSpeed, 360.0f);
}

```


Then for other animations I made a state machine system where I switch them on/off with my bools.

![image](https://github.com/user-attachments/assets/98a39e88-4e9c-4345-90d9-9dc111b8e859)

I changed the booleans in my script. Before showing you the boolean logic, I want to mention a problem I had. To detect if the car had hit something, I added a box collider as usual. However, whenever I pressed the play button in Unreal, the collision box just disappeared. After hours of trying, I gave up and decided to create my own 'collision box' by using a raycast from the car to detect if it hit something.

  
```csharp

void ACharacterInput::CrashDetection()
{
	const float StartHeightOffset = 15.0f;
	const float TraceDistance = 50.0f;
	const float FeedbackIntensity = 0.4f;
	const float FeedbackDuration = 0.3f;
	const float SpeedThresholdForScreenShake = 600.0f;

	bool bWasHitPreviously = bHitSomething;
	FVector Start = CarMesh->GetComponentLocation() + FVector::UpVector * StartHeightOffset;
	FVector End = Start + Direction * TraceDistance;

	FHitResult Hit;
	FCollisionQueryParams CollisionParams;
	CollisionParams.AddIgnoredComponent(CarMesh);

	bHitSomething = GetWorld()->LineTraceSingleByChannel(Hit, Start, End, ECC_Visibility, CollisionParams);
	UKismetSystemLibrary::DrawDebugLine(GetWorld(), Start, End, FColor(255, 0, 0));

	if (bHitSomething != bWasHitPreviously)
	{
		if (bHitSomething)
		{
			AActor* HitActor = Hit.GetActor();
			if (HitActor && HitActor->ActorHasTag(FName("Fence")))
			{
				if (AnimInstance)
				{
					AnimInstance->isCrashing = true;
				}
				bHasCrashed = true;

				APlayerController* PlayerController = GetWorld()->GetFirstPlayerController();
				if (PlayerController)
				{
					PlayerController->PlayDynamicForceFeedback(FeedbackIntensity, FeedbackDuration, true, true, true, true);
				}

				if (MovementComponent && MovementComponent->CurrentSpeed > SpeedThresholdForScreenShake)
				{
					StartScreenShake(2, 0.1);
				}
			}
		}
		else
		{
			if (AnimInstance)
			{
				AnimInstance->isCrashing = false;
			}
			bHasCrashed = false;
		}
	}
}

```
</details>

Thank you for reading!

itch.io: https://yrgo-game-creator.itch.io/hellracer
