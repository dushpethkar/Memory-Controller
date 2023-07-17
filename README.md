"# Memory-Controller"
----------------------------------------------------------------------------------

-- Design Name:	 Memory Controller
-- Module Name:    Top_AMBA - Behavioral 
-- Project Name: 	 lDesign and Implementation Of AMBA-AHB based Memory Controller
----------------------------------------------------------------------------------
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.STD_LOGIC_ARITH.ALL;
use IEEE.STD_LOGIC_UNSIGNED.ALL;
use work.MyDataTypes.all;
use IEEE.NUMERIC_STD.ALL;


entity Top_AMBA is

    PORT( 
				HCLK    : In  STD_LOGIC;
            HRESETn : In  STD_LOGIC
      );
			 
end Top_AMBA;

architecture Behavioral of Top_AMBA is

COMPONENT CoreAHB
GENERIC (
          numCore     : integer range 0 to Ncores-1:=0
        );
        PORT(
			  HCLK    : IN  std_logic;
           HRESETn : IN  std_logic;
	        HREADY_M : IN  std_logic;
           HRESP_M :IN  std_logic_vector(1 DOWNTO 0);
           HGRANT : IN  std_logic;
           HTRANS_S : IN  std_logic_vector(1 DOWNTO 0);
           HWDATA_S : IN  std_logic_vector(NumDataBits-1 downto 0);
           HMASTER : IN  std_logic_vector(HADDRbits-1 downto 0);
           HTRANS_M : OUT  std_logic_vector(1 downto 0);
           HWDATA_M : OUT  std_logic_vector(NumDataBits-1 downto 0);
           HBUSREQ :OUT  std_logic;
           HREADY_S : OUT  std_logic;
           HRESP_S : OUT  std_logic_vector(1 downto 0)
	     ); 
END COMPONENT;

COMPONENT arbiterAHB
PORT(
		     HCLK    : IN  std_logic;
           HRESETn : IN  std_logic;
	        HREADY  : IN  STD_LOGIC;
			  HBUSREQ : IN  std_logic_vector(Ncores-1 downto 0);
           HGRANT  : OUT  std_logic_vector(Ncores-1 downto 0);
           HMASTER : OUT  std_logic_vector(HADDRbits-1 downto 0)
);
END COMPONENT;

COMPONENT decoderAHB
PORT (
	  SEL: IN std_logic_vector(HADDRbits-1 downto 0);
	  OUTPUT: OUT std_logic_vector(Ncores-1 downto 0)
);
END COMPONENT;

--internal Signals---
signal HTRANS       :STD_LOGIC_VECTOR(1 DOWNTO 0);
signal HTRANS_VECT  :twobit_vect(Ncores-1 DOWNTO 0);
signal HWDATA		  :STD_LOGIC_VECTOR(NumDataBits-1 DOWNTO 0);
signal HWDATA_VECT  :data_vect(Ncores-1 DOWNTO 0);
signal HREADY       :STD_LOGIC;
signal HREADY_VECT  :STD_LOGIC_VECTOR(Ncores-1 DOWNTO 0);
signal HRESP        :STD_LOGIC_VECTOR(1 DOWNTO 0);
signal HRESP_VECT   :twobit_vect(Ncores-1 DOWNTO 0);
---------

--Arbiter signals---
signal HBUSREQ            :STD_LOGIC_VECTOR(Ncores-1 DOWNTO 0);
signal HGRANT             :STD_LOGIC_VECTOR(Ncores-1 DOWNTO 0);
signal HMASTER            :STD_LOGIC_VECTOR(HADDRbits-1 DOWNTO 0);
signal HMASTER_delayed    :STD_LOGIC_VECTOR(HADDRbits-1 DOWNTO 0);
-------------

begin

----HREADY DECODER------
HREADY  <= '1' when not(HREADY_VECT)=0 ELSE '0';


----HWDATA DECODER-----
HMASTER_delay: process(HCLK)
begin
    if(HCLK='1' and HCLK'event) then
	 HMASTER_delayed<=HMASTER;
	 end if;
	 end process;
	 
HWDATA <= HWDATA_VECT(conv_integer(HMASTER_delayed));
-----------

-----HRESP DECODER----
HRESP<="00";
-- Two bits are used, but ut would be enough with 1 bit to indicate "OKAY" and "ERROR".

-- In this project, only "OKAY" value is assigned ("00"). "ERROR" value is due to physical failure of the bus.
--------------

------HTRANS DECODER-----
HTRANS <= HTRANS_VECT(conv_integer(HMASTER));
-----------

------------------------------------------------
---     n cores instantiations               ---
------------------------------------------------

Cores : for n in 0 to Ncores-1 generate
   Core_n: CoreAHB
	GENERIC MAP (
			numCore => n
	)
	
	PORT MAP(
		HCLK => HCLK,
		HRESETn => HRESETn,
		HTRANS_M => HTRANS_VECT(n),
		HWDATA_M => HWDATA_VECT(n),
		HREADY_M => HREADY,
		HRESP_M => HRESP,
		HBUSREQ => HBUSREQ(n),
		HGRANT => HGRANT(n),
		HTRANS_S => HTRANS,
		HWDATA_S => HWDATA,
		HREADY_S => HREADY_VECT(n),
		HRESP_S => HRESP_VECT(n),
		HMASTER => HMASTER
	);
	end generate;
------------------------------------

arbiterAHB_0: arbiterAHB PORT MAP(
	HCLK => HCLK,
	HRESETn => HRESETn,
	HREADY => HREADY,
	HBUSREQ => HBUSREQ,
	HGRANT => HGRANT,
	HMASTER => HMASTER
);	

end Behavioral;


