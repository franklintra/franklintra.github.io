# Cheatsheet VHDL

**<u>DISCLAIMER</u>**: This VHDL cheatsheet <u>will not replace</u> the material given in class. It was inspired by this [readme](https://github.com/ismailelbadawy/vhdl-cheat-sheet).

However, this cheatsheet might help you learn subtlety of VHDL by learning on examples that were used for lab session 8/9.

### Libraries

1. Standard library
  
  ```vhdl
  Library ieee;
  ```
  
2. Standard logic library
  
  ```vhdl
  use ieee.std_logic_1164.all;
  ```
  
3. Math library
  
  ```vhdl
  use ieee.numeric_std.all;
  ```
  

### Entities

1. Non-generic entity example: Debounce
  
  ```vhdl
  -- the imported libraries are only given in the very first example
  -- to not -Kluter- clutter the document
  LIBRARY ieee;
  USE ieee.std_logic_1164.ALL;
  USE ieee.numeric_std.ALL;
  USE work.utils.ALL;
  
  ENTITY debounce IS
      PORT (
          clk : IN STD_LOGIC;
          n_reset : IN STD_LOGIC;
          button_in : IN STD_LOGIC;
          button_out : OUT STD_LOGIC);
  END debounce;
  ```
  
2. Generic entity example : Timer with default count value
  
  ```vhdl
  ENTITY timer IS
      GENERIC (count : INTEGER := 8);
      PORT (
          clk : IN STD_LOGIC;
          n_reset : IN STD_LOGIC;
          reset : IN STD_LOGIC;
          flag : OUT STD_LOGIC);
  END timer;
  ```
  

## Architecture

1. Default example of an empty architecture
  
  ```vhdl
  ARCHITECTURE rtl OF ENTITY timer IS
  -- Where signals/components are declared
  SIGNAL count : INTEGER := 0;
  
  BEGIN
  -- Where the actual logic takes place with or without process
  END rtl;
  ```
  
2. Using another entity inside of the architecture: debounce example
  
  ```vhdl
  ARCHITECTURE rtl of ENTITY debounce IS
      -- all signals used below have to be declared
      -- (clk, n_reset, s_reset, s_clear)
      -- not done here for clarity
      
      -- Component is basically re-declaring the entity
  	COMPONENT timer
          -- this is optional if we want to keep the default values
          -- but then we should remove the GENERIC MAP (a=>b) call
  		GENERIC (count : INTEGER);
  			PORT (
  				clk : IN STD_LOGIC;
  				n_reset : IN STD_LOGIC;
  				reset : IN STD_LOGIC;
  				flag : OUT STD_LOGIC);
  	END COMPONENT;
      
      -- 
  BEGIN
  
  timer_instance : timer GENERIC MAP (count => 8)
      --generic map takes CONSTANTS ONLY
      --Ommit if the component is not generic
      PORT MAP (
      -- (timer in/out) => (architecture signals)
      clk => clk,
      n_reset => n_reset,
      reset => s_reset,
      flag => s_clear
  );
  
  END rtl;
  ```
  

## Processes

A process is **executed sequentially** rather than the usual concurrent execution.

1. Example with the timer_ctrl
  
  ```vhdl
  ARCHITECTURE rtl OF ENTITY debounce IS
  
  BEGIN
  -- Where the actual logic takes place with or without process
  
      -- This process is the controller of the imported entity
      -- It will be run sequentially every time there is a change in
      -- either clk or n_reset (sensitivity list)
      timer_ctrl : PROCESS (clk, n_reset) IS
      -- Where process variables / signal go
      BEGIN
          IF n_reset = '0' THEN
              -- async reset logic
              reset <= '1';
          ELSIF RISING_EDGE(clk) THEN
              -- timer_ctrl logic (when to count)
          ELSE
              report "In the else (doesn't use THEN)"
          END IF;
      END PROCESS timer_ctrl;
  
  END rtl;
  ```
  

### Assignments

We can either assign values to signal or variables. So let's start by diving into their differences:

##### Use a SIGNAL or a VARIABLE

|     | Signal | Variable |
| --- | --- | --- |
| **Assignment** | Uses "<=" | Uses ":=" |
| **Scope** | The entire architecture | Limited to the process or block where declared |
| **Assignment Timing** | <u>Takes effect at the end of the current simulation time unit</u> : If you assign twice a signal in a process, the first assignment is ignored. | <u>Takes effect immediately</u> |
| **Propagation** | Can be propagated across different processes | Can't be propagated across different processes |
| **Storage** | <u>Persist values between different invocations of the process</u> | Does not persist values |

1. Classic assignment example
  
  ```vhdl
  ARCHITECTURE myArch OF ENTITY test IS
      SIGNAL count : Integer := 0;
      -- the notation (OTHERS => '0') is to set all values in array
      -- the notation 15 downto 0 is the numerotation of the bits
      -- we can then call vector(15) to get the msb of the array
      SIGNAL vector : STD_LOGIC_VECTOR (15 downto 0) := (OTHERS => '0'); 
      SIGNAL std : STD_LOGIC := '0'; -- := for initial value
  BEGIN
  
  count <= 12; -- <= for signals
  
  prcs : PROCESS(clk) IS
  VARIABLE process_var : STD_LOGIC := '0';
  BEGIN
  	IF rising_edge(clk) THEN
  		IF process_var = '0' THEN
  			process_var := '1'; -- := for variables
  		ELSIF process_var = '1' THEN
  			process_var := '0';
  		ELSE
  			report "Impossible case";
  		END IF;
  	END IF;
  END prcs;
  
  vector <= "0000000000000000"; -- double quotes "" for vector
  std <= '1';                   -- simple quotes '' for std
  vector <= (OTHERS => '1');    -- whole vector is 1
  
  END myArch;
  ```
  
2. Conditional assignment example
  
  I will not in this example show the difference between signal and variables but just for a signal (The reader can infer the rest)
  
  ```vhdl
  ARCHITECTURE myArch OF ENTITY test IS
      SIGNAL std : STD_LOGIC_VECTOR := '0';
      SIGNAL opposite_std : STD_LOGIC_VECTOR; -- no default values
  BEGIN
  
  changeWithClock : PROCESS(clk) IS
  BEGIN
      IF rising_edge(clk) THEN
          IF std = '0' THEN
              std <= '1';
          ELSIF std = '1' THEN
              std <= '0';
          END IF;
      END IF;
  END changeWithClock;
  
  opposite_std <= '1' WHEN std = '0'
                  ELSE '0' WHEN std = '1'
                  ELSE 'U'; -- 'U' stands for unknown
  
  END myArch;
  ```
  

### Type conversions and types

![](https://media.cleanshot.cloud/media/1430/fJ7VMVAivqr7gnxWicfMmsZqo49o5inNnM30WfF6.jpeg?Expires=1684877396&Signature=agf~qznSllhqePOCqipcT5znQ2c6z3AalRjLK6wakL~UKw5xo94Fj5029ChcmSXJlTho-G~rCCRwb3ZCmRLS9LXnUTD9An5gp5JV-AKxC92Y7uzmv-jRTr5iEde-OolJ26PBn~ioWNgQmy88BlDsHDtPMF3PIpiKt5tspJ27VsUfOIeXqs4HAOKRbkqgzbzOaenNok3D8EjcJdljg3WQB2R5D-KBCnvegMzl~ArvBy-h5Rzshh-IJpJvQeu4QwkWRtSsutQhjqO4CgexN-B~PZNWGPwIwyf9zv1x3nWPtTAhzl~yXK1tVfhNAfqvw1ywR15bqWsgBuo8RGqr9ZRkZg__&Key-Pair-Id=K269JMAT9ZF4GZ)

### Operators

##### Mathematical

| Operator | Function |
| --- | --- |
| **  | Power |
| rem | Remainder |
| mod | Modulus |
| /, *, -, + | Self-explanatory |
| abs | Absolute value |

##### Logical Operators

| Operator | Function |
| --- | --- |
| not | !   |
| and | &&  |
| or  | \|  |
| xor | A xor B |
| nand, nor, xnor | Self-explanatory |

##### Relational Operators

| Operator | Function |
| --- | --- |
| =   | ==  |
| /=  | !=  |
| <, >, <=, >= | Self-explanatory |

### Code snippets

#### Timer

```vhdl
LIBRARY ieee;
USE ieee.std_logic_1164.ALL;
USE ieee.numeric_std.ALL;

ENTITY timer IS
    GENERIC (count : INTEGER := 8);
    PORT (
        clk : IN STD_LOGIC;
        n_reset : IN STD_LOGIC;
        reset : IN STD_LOGIC;
        flag : OUT STD_LOGIC);
END timer;

ARCHITECTURE rtl OF timer IS
    SIGNAL counter : UNSIGNED(63 downto 0);
BEGIN
    count_process : PROCESS (clk, n_reset) IS
    BEGIN
        IF n_reset = '0' THEN -- n_reset is active low
            counter <= (OTHERS => '0'); -- reset the counter
        ELSIF rising_edge(clk) THEN
            IF reset = '1' THEN
                counter <= (OTHERS => '0'); -- reset the counter
            ELSIF reset = '0' THEN -- start counting
                counter <= (counter + 1) mod count;
            END IF;
        END IF;
    END PROCESS count_process;
    flag <= '1' WHEN counter = count - 1 ELSE '0';
END rtl;
```

### Finite state machine (Traffic controller)

```vhdl
library ieee;
use ieee.std_logic_1164.all;
use ieee.numeric_std.all;

entity traffic_light_controller is
    Port (
        clk : in std_logic;
        reset : in std_logic;
        pedestrianButton : in std_logic;
        light : out std_logic_vector (2 downto 0)
    );
end traffic_light_controller;

architecture Behavioral of traffic_light_controller is
    type state_type is (RED, YELLOW, GREEN);
    signal state, next_state : state_type;
begin
    process(clk, reset)
    begin
        if reset = '1' then
            state <= RED;
        elsif rising_edge(clk) then
            state <= next_state;
        end if;
    end process;

    process(clk)
    begin
        IF rising_edge(clk) THEN
            case state is
                when RED =>
                    -- should implement a timer for how long red has been
                    next_state <= GREEN;
                when GREEN =>
                    IF pedestrianButton = '1' THEN
                        next_state <= YELLOW;
                    ELSE
                        next_state <= GREEN;
                    END IF;
                when YELLOW =>
                    next_state <= RED;
                when others =>
                    next_state <= RED;
            end case;
        END IF;
    end process;

light <= "001" WHEN state <= GREEN
               ELSE "010" WHEN state <= YELLOW
               ELSE "100"; -- red or others
-- this light is async to the states (real-time changes)
end Behavioral;
```
