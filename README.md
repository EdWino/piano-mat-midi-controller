//piano-mat-midi-controller
//code in progress for a arduino controlled midi piano key floor mat
// Pin Definitions
// Rows are connected to
const int row1 = 5;
const int row2 = 6;

// The 74HC595 uses a serial communication 
// link which has three pins
const int clock = 9;
const int latch = 10;
const int data = 11;


uint8_t keyToMidiMap[24];

boolean keyPressed[24];

int noteVelocity = 127;


// use prepared bit vectors instead of shifting bit left everytime
int bits[] = { B00000001, B00000010, B00000100, B00001000, B00010000, B00100000, B01000000, B10000000, B00000011, B00000110, B00001100, B00011000, B00110000, B01100000 };


// 74HC595 shift to next column
void scanColumn(int value) {
	digitalWrite(latch, LOW); //Pulls the chips latch low
	shiftOut(data, clock, MSBFIRST, value); //Shifts out the 8 bits to the shift register
	digitalWrite(latch, HIGH); //Pulls the latch high displaying the data
}

void setup() {
	
	// Map scan matrix buttons/keys to actual Midi note number. Lowest num 41 corresponds to F MIDI note.
	keyToMidiMap[0] = 53;
	keyToMidiMap[1] = 55;
keyToMidiMap[2] = 57;
keyToMidiMap[3] = 59;
keyToMidiMap[4] = 60; 
keyToMidiMap[5] = 62;
keyToMidiMap[6] = 64;
keyToMidiMap[7] = 65;
keyToMidiMap[8] = 67;
keyToMidiMap[9] = 69;
keyToMidiMap[10] = 71;
keyToMidiMap[11] = 72;
keyToMidiMap[12] = 74;
keyToMidiMap[13] = 76;

keyToMidiMap[0 + 14] = 54;
keyToMidiMap[1 + 14] = 56;
keyToMidiMap[2 + 14] = 58;
keyToMidiMap[4 + 14] = 61;
keyToMidiMap[5 + 14] = 63;
keyToMidiMap[8 + 14] = 66;
keyToMidiMap[9 + 14] = 68;
keyToMidiMap[10 + 14] = 70;
keyToMidiMap[12 + 14] = 73;
keyToMidiMap[13 + 14] = 75;

	// setup pins output/input mode
	pinMode(data, OUTPUT);
	pinMode(clock, OUTPUT);
	pinMode(latch, OUTPUT);

	pinMode(row1, INPUT);
	pinMode(row2, INPUT);

    Serial.begin(31250);

	delay(1000);

}

void loop() {

	for (int col = 0; col < 14; col++) {
		
		// shift scan matrix to following column
		scanColumn(bits[col]);

		// check if any keys were pressed - rows will have HIGH output in this case corresponding
		int groupValue1 = digitalRead(row1);
		int groupValue2 = digitalRead(row2);

		// process if any combination of keys pressed
		if (groupValue1 != 0 || groupValue2 != 0) {

			if (groupValue1 != 0 && !keyPressed[col]) {
				keyPressed[col] = true;
				noteOn(0x91, keyToMidiMap[col], noteVelocity);
			}

			if (groupValue2 != 0 && !keyPressed[col + 14]) {
				keyPressed[col + 14] = true;
				noteOn(0x91, keyToMidiMap[col + 14], noteVelocity);
			}

		}

		//  process if any combination of keys released
		if (groupValue1 == 0 && keyPressed[col]) {
			keyPressed[col] = false;
			noteOn(0x91, keyToMidiMap[col], 0);
		}

		if (groupValue2 == 0 && keyPressed[col + 14]) {
			keyPressed[col + 14] = false;
			noteOn(0x91, keyToMidiMap[col + 14], 0);
		}


	}

}


void noteOn(int cmd, int pitch, int velocity) {
  	Serial.write(cmd);
	Serial.write(pitch);
	Serial.write(velocity);
}

