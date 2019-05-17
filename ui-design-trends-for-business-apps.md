# On UI design trends for business apps

Note: I will use the word app in the following to mean: 'program', 'application', 'web app', 'store app' and 'software service'


## What is UI?

UI stands for User Interaction/Interface: the means for a user of a program/app. to influence the program/app's behaviour and also interpret a program/app's results


## Historical Evolution

- Pure batch apps: no interaction beyond setting startup configuration and interpreting final results:
  - Startup config via command line params, job params and files: config. files and input/output files
  - Result interpretation by looking at output files, including output log/syslog/stdout/stderr and an exit code number
  - Users don't monitor the app's execution as there is no possibility to interact with it once started and until it has ended
  - Examples: mainframe batch jobs, UNIX command line utils, scheduled jobs/tasks, ...

- Console/command line user-assisted apps: interaction by line-based textual request&response:
  - The app sends 'questions' to a console that is visible to user and then wait for a response
  - The user does monitor the app's execution and answers 'questions' by typing in a response (and typically pressing ENTER to indicate that the response is ready)
  - The app then interprets the response and based on this interpretation can alter its behaviour, possibly leading to more request/response cycles
  - These kinds of apps already have the notion of a stateful session since questions sent by the app will depend on previous questions and responses
  - Examples: command line install/setup programs, command line ftp/sftp/telnet, ...
  
- Console/command line dialog apps: technically the same as the previous category but different in that the apps' functioning is controlled completely by the user
  - While technically the app still gives control to the user by asking a 'question' and waiting for a 'response', the feel of the apps is exactly the reverse: it is the user that 'asks' questions and the app that 'responds'
  - Examples: command line calculator, command line games, ...

- TUI/Text-based/Terminal based dialog apps: interaction by manipulation (with keyboard and mouse) of a rows-by-columns 2D surface
  - The user is in full control typically and the app is structured to respond to specific user requests
  - The app fills an initial 'terminal' screen in a way that asks the user to fill in certain parts and then inform the app to do something
  - The app can use 'attributes' to make certain parts of the terminal look distinctive (e.g. colors, borders) and also behave distinctive (e.g. can type inside or not)
  - There is often the notion of a cursor: an active 'cell position' in the row x columns grid of terminal positions
  - Communication sometimes involves the app monitoring every keystroke (and cursor/mouse movement) and sometimes the app. receiving a full data buffer corresponding to the current cursor/mouse position + last keystroke code + full rows x columns data buffer
  - Examples: 
    - IBM mainframe green-screen apps running under CICS, IMS, IDMS/DC: 3270, 5250 -- full buffer comm.
    - BS2000 mainframe green-screen apps running under UTM: 9750  --- full buffer comm.
    - UNIX terminal/curses apps: VT100, VT2200: vi, emacs, ... --- event comm.
    - Windows console apps: LOTUS-1-2-3 --- event comm.
    
- GUI based dialog apps: conceptually the same as TUI but now the 2D surface doesn't consist of a finite number of cells that can each contain a single character/symbol, but by pixel cells
  - The apps are always structured to respond to 'GUI' events, which can be keystrokes, mouse movements and/or mouse button presses (and even audio or tactile inputs)

- Distributed Client/Server apps: app functionality is split between two parts that live on different machines separated by a network
  - Console, TUI and GUI apps can all be distributed
  - Examples: Database client+server, mail client+server, ...

- Web apps: they this really is a categorization purely on technology grounds (distributed + using HTTP)
  - These are always distributed with client and server parts
  - From a client 'intelligence' point of view the can behave like simple Console/TUI based apps that merely allow for simple data-entry without any client side logic beyond look&feel
  - From a data-exchange point of view, they can follow the event based pattern or a full buffer comm. pattern
  - Examples: mobile app store apps, SOA web services + clients, REST web services + clients, interactive websites, single-page web-apps, ....
  
 
## How to Categorize?  

- Distributed or not: separate processes possibly separated by a network,  or 1 process on 1 machine
- Interactive or batch: interactive means that the app. is written to respond to user inputs and the user is perceived in control of what the app is doing
- Dumb terminal or distributed processing/intelligence: 
  - A dumb terminal doesn't contain any real application logic and only concerns itself with the look&feel aspects. It also doesn't need to know the app itself as it's functionality is restricted to input/output handling tasks that can be shared by different apps. Examples: 3270, VT100, browser (? or not)
  - Aistributed intelligence means that both client and server contain important processing that is not restricted to look&feel aspects
- Granularity of client/server interaction: small events <> full request/response buffers
