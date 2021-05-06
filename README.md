<!-- wp:paragraph {"align":"justify"} -->
<p class="has-text-align-justify">The <a href="https://www.nxp.com/docs/en/data-sheet/MC9S08AW60.pdf">AW60 microcontroller unit</a> (MCU) is capable to interpreting C and Assembly Programming languages. However, even though programming the micromouse in Assembly is possible and an option, it is not the easiest, most effective, or fastest method to programme multiple processes as the micromouse is expected to do. Due to this C is used for its ability to run multiple processes quickly, and effectively with fewer lines of code.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"align":"justify"} -->
<p class="has-text-align-justify">To compile the C and port code the <a href="https://www.nxp.com/design/software/development-software/codewarrior-development-tools/codewarrior-legacy/codewarrior-for-mcus-eclipse-ide-coldfire-56800-e-dsc-qorivva-56xx-rs08-s08-s12z-11-1:CW-MCU10?tab=Documentation_Tab">Code Warrior 10.7 </a>(Eclipse) Development studio is used since it supports AW60 hardware interpretation. Code warrior allows for effective debugging of MCU via its debugger. The debugger allows the programmer to analyse specific processes and variables to insure the expect values are being moved in and out the various MCU registers. Code warrior also allows the programmer to view the disassembled (assembly) version of your source code giving you more you to optimise your program.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"align":"justify"} -->
<p class="has-text-align-justify">The Team Delta micromouse uses a modified version of the provided Micromouse source code which was developed by Dr. Timothy Davies. This Software allows the micromouse to perform its most important operation, to control the various sensors and motors connect to it.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"align":"justify"} -->
<p class="has-text-align-justify">The flowchart below illustrates the sequence of processes that enable the microcontroller to interact with the outside world, i.e. the motors and sensors. The raw version of the code can be found in the link below.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"align":"center","id":1224,"sizeSlug":"large","linkDestination":"none"} -->
<div class="wp-block-image"><figure class="aligncenter size-large"><img src="https://teamdelta2021.files.wordpress.com/2021/05/screenshot-2021-05-06-at-9.46.37-pm.png?w=908" alt="" class="wp-image-1224"/><figcaption>Flow Chart of the main function</figcaption></figure></div>
<!-- /wp:image -->

<!-- wp:paragraph {"align":"justify"} -->
<p class="has-text-align-justify">The main function performs the tasks of, disabling the <a href="https://en.wikipedia.org/wiki/Watchdog_timer">watchdog</a> to prevent having to periodically "<a href="https://demmel.com/ilcd/help/FeedWatchdog.htm">feed</a>" it and an unwanted reset, enabling the external quartz crystal, initialising the timer interrupt which is periodically by the two motors depending on the prescaler value and the set timer modulus on timer module 1, in the case of this program we use a modulus of <strong>20000 (decimal)</strong> which gives a 10ms period. Two timer channels are configured channel 1 and 2. The function then falls into an infinite loop (for loop), this is done so that the program runs for as long as the battery voltage allows. TPM2 C0 and C1 are enabled for edge detection of the feedback from the two encoder outputs.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"align":"justify"} -->
<p class="has-text-align-justify">Within the <strong>for</strong> loop the variable <strong>DRIVE</strong> is set to <strong>FWD</strong> which sets the two bits 0 and 2 high, to move the robot forward or backwards if the connects were accidentally reversed. The loop then calls the obstacle avoidance function which checks for inputs from the two sensors i.e. infrared and collision sensors. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"align":"justify"} -->
<p class="has-text-align-justify">Microcontroller goes into its <a href="https://canvas.swansea.ac.uk/courses/15962/files/1307218/preview">interrupt service routine</a> and generates a PWM signal to control the motors. Since interrupts work on a basis of vector priority unlike polling, lower value interrupts are called first in the case of this program interrupt 6. The flowchart below illustrates the sequence in which the interrupts will occur.  </p>
<!-- /wp:paragraph -->

<!-- wp:image {"align":"center","id":1096,"width":350,"height":713,"sizeSlug":"large","linkDestination":"none"} -->
<div class="wp-block-image"><figure class="aligncenter size-large is-resized"><img src="https://teamdelta2021.files.wordpress.com/2021/05/screenshot-2021-05-06-at-8.04.34-pm.png?w=502" alt="" class="wp-image-1096" width="350" height="713"/><figcaption>Flow chart of the Interrupt Service Routine used by Delta2021.c</figcaption></figure></div>
<!-- /wp:image -->

<!-- wp:paragraph {"align":"justify"} -->
<p class="has-text-align-justify">The first two interrupts called , i.e. Interrupts 6 and 7 set the motors into free wheel mode, interrupt 11 turns the motors on as define in the variable <strong>DRIVE</strong>, then calls the <strong>speedcon()</strong> function to begin reading the speed of the motors to control, at the moment no changes have been registered yet, it then finally checks if the counter variable is equal to zero and if not, decrements by one.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"align":"justify"} -->
<p class="has-text-align-justify"> Interrupts 12 and 13 intended to record the changes in the speed of the two motors via the encoders, This is done for speed control. first resets <strong>TPM2C0_CH0F </strong>and <strong>TPM2C1_CH1F </strong>, then set the the current PWM as <strong>NEW_LEFT/RIGHT</strong> of each motor to the value of <strong>TPM2C0V</strong> and <strong>TPMC1V</strong>.the current PWM is subtracted from the older PWM (<strong>OLD_LEFT/RIGHT</strong>) to find the difference which stored in the variable <strong>DIFF_LEFT/RIGHT</strong>. and finally checks to see if the DISTANCE variable is zero, and  if not decrements it by one.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":1055,"sizeSlug":"large","linkDestination":"none"} -->
<figure class="wp-block-image size-large"><img src="https://teamdelta2021.files.wordpress.com/2021/05/screenshot-2021-05-06-at-6.03.37-pm.png?w=1024" alt="" class="wp-image-1055"/><figcaption>Flow Chart of the Obstacle Avoidance function</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph {"align":"justify"} -->
<p class="has-text-align-justify">The <strong>obstacle_avoidance</strong> functions tells the micromouse how to response to a collision or an obstacle sensed by either the <a href="https://teamdelta2021.files.wordpress.com/2021/03/collision-sensor-documentation-4.pdf">collision sensor</a> or the<a href="https://teamdelta2021.com/ir-sensor/"> infrared sensor. </a>The flowchart above shows the sequence of statements performed when this function is called. The function first checks if either micro switch has collided with an obstacle if not it continues down the line otherwise either function revleft or revright are called.  The infrared sensor operation is slightly more complicated than the collision sensor, since it has two nested if statements to determine the proper response if either the left or the right infrared sensor detects an object or obstacle, which is to call iravoidl or iravoidr respectively. if no input is received from sensor the function is exited.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"align":"justify"} -->
<p class="has-text-align-justify">As is with programming there are various ways to do a task, an alternative to using if statements to detect sensor inputs would be to use an edge enabled keyboard interrupt, however you would have to disable the interrupt from within the function to ensure that no interrupts are missed while performing the task.</p>
<!-- /wp:paragraph -->
