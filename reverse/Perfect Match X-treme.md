## Skills involved: Reversing Unity Game

This challenge is clear about what to do but there are a lot of ways.

## Solution:

We are given a Build folder with `PerfectMatch.exe` and amongst other files, `UnityPlayer.dll`.

The game is a simple version of [Fall Guy's Perfect Match](https://www.youtube.com/watch?v=edifg0uMzxU), but the third stage cannot be passed no matter what. That need to change.

Here is a quick guide to [Unity game hacking](https://github.com/imadr/Unity-game-hacking):

![image](https://user-images.githubusercontent.com/114584910/193578847-0ae7cdc3-4f15-4e7a-82ee-9497c4ccbcbe.png)

Since I already have dnSpy, I can just load the `Assembly-CSharp.dll` in it. This way I do not need Unity editors.

<img width="671" alt="scrnsht2" src="https://user-images.githubusercontent.com/114584910/193580797-5e4197c5-f4c5-4cd4-8bc2-ddd9068e1e7b.png">

I knew breakpoints can be set in it for single binaries, and the local variable values can be changed in dnSpy but I'm not familiar with dnspy and C# in general. I was hinted that that I can edit the decompiled source and compile it back then.

- I was able to bypass the -20 y value check for gameover very easily (*HeightCheck.Update()*), but I found out the flags are hidden after the Scoreboards and I can't see them.
- I could disable gravity and jump towards both scoreboards while dashing: (*MoveBehaviour.MovementManagement*)
  - This is not a very stable method and I always got stuck
  - I can end up very high above and can't see the flags
  - Sometimes I was lucky that I could move horizontally freely even after jumping away, I cannot repeat it

<img width="421" alt="scrnsht1" src="https://user-images.githubusercontent.com/114584910/193581686-a783efba-0ae4-4df7-89bb-8a8a1797d9e5.png"> (mirrored)
<img width="421" alt="scrnsht3" src="https://user-images.githubusercontent.com/114584910/193604404-59ac0915-ce44-454a-8abe-3b8afe755978.png">

I finally disabled gravity and set the game to win at 1 round (*GameManager.IncreaseRound*). This way my height is will not be very large and the flag pieces are still visible.

P.S. I have to Save Module after Compiling the Class/Methods and select **Mixed mode**, otherwise the classes cannot be edited again. I'm not sure what that meant.

P.S.2. This challenge can be trivially solved by grepping `SEKAI` and finding the strings in `level0` - modding the game is much more enjoyable though.
