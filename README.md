# HellRacer-Workshop

Hello HellRacer is a race game where you play against your ghost which is the best time that is saved in your game. 
We made the game in unreal engine with c++. I had watched some videos for learning c++ but i cant say that  i could c++ before. It was mostly the knowledge i had before from unity and chat gbt that helped me. 

My area of responsibility was car movement. First I had no idea where to start then with some youtube vieos i made a simple movement class based on Character. After 2 week i made a simple movement but i wanted to try other methods too. One of them was make a new movement class based on pawn. But i didnt like any other then character movement because the other ones didnt had any physic. I could of course create a physic logic but I thought it would be too stressfull to do that and the car movement in just 8 weeks. Thats why i deleted all new methods and went back to the first one. 

We decided as programmer to make a main class for the car and attach all other scripts to that, movement, drift, particle and other stuff. I made the Car movement seperatly, functions for each thing. All my logic was based on characterMovement values. But i had a big problem. Whenever the car was in move and you relased the accelerate button and rotated the car, it kept the old rotation and moved that direction which gave a slide effect. That was not what I wanted but I couldnt find the problem either. Instead I kept the input at 1 and made a separate function for brake. 

For animations I didnt really knew where to start but after some exploration I found a workflow. I made animinstances.
