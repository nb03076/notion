# LPTIM

![Untitled](LPTIM%20a5e910f6dce943ef8e7ceac7a8361467/Untitled.png)

그냥 Timer랑 솔직히 기능만 줄어들었다 뿐이지 필요한건 다있고 동작방식도 약간 비슷한듯..

근데 LSE 처럼 다른 클럭 소스도 이용가능해서 좋음. 그리고 STM32WB에서는 oneshot 타이머 기능도 제공함(LPTIM_CR_SNGSTRT)

glitch filter를 이용해서 잘못 들어온 펄스를 죽여버리게 하는듯(이 부분은 좀더 봐야할듯)