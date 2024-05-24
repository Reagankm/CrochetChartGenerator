import java.util.ArrayList;
import java.util.List;

public class Main {

    public static void main(String[] args) {
        final String[] rows = {"Row 35: CH3. 2DC in first stitch. DC in next 6 stitches. {CH1, SK1, DC in next 11 stitches.} Repeat  between { } 9 times. CH1, SK1, DC in next 7 stitches. In CH3 space make 2DC, CH3, 2DC. DC in  next 7 stitches. {CH1, SK1, DC in next 11 stitches.} Repeat between { } 9 times. CH1, SK1, DC in  next 6 stitches. 3DC in last stitch."};

        final Tokenizer tokenizer = new Tokenizer(rows[0].substring(rows[0].indexOf("in first stitch.") + "in first stitch.".length()));
        while (tokenizer.hasNext()) {
            //System.out.println(tokenizer.nextInstruction());
            System.out.println(tokenizer.parseInstruction(tokenizer.nextInstruction()));
        }

    }


}

class Tokenizer {
    private String input;
    private int position;

    public Tokenizer(String input) {
        this.input = input;
        this.position = 0;
    }

    public boolean hasNext() {
        return position == 0 || position < input.length();
    }
    // So, we build up tokens then parse the tokens
    // When parsing, look for DC or SC, qty (could be before or after), and location

    public String parseInstruction(String instruction) {
        // iterate over instruction chars looking for either DC, SC, or an int
        final char[] chars = instruction.toCharArray();
        int qty = -1;
        for (int i = 0; i < instruction.length(); i++) {
            char curr = chars[i];

            if (Character.isDigit(chars[i])) {
                int start = i;
                while (i < chars.length && Character.isDigit(chars[i])) {
                    i++;
                }
                qty = Integer.parseInt(instruction.substring(start, i));
                // Decrement i because current i is not an int and the loop will increment it again, so the current i will never be processed
                i--;
            } else {
                // DC = x
                // 3DC in stitch = w
                // 2 DC in stitch = v
                // CH = o
                // sk = '' (basically always comes after a CH1 so nothing needs to be done here)
                Character symbol = null;

                // I was going to consume char by char to find tokens, but is it instead better to do a bunch of regex/indexOf searches in the string instead?
                // indexOf DC, SC, CH, SK
                // match ints, are they before or after
                // ... :/

                // identifying token should set some symbol property to match?
                switch (curr) {
                case 'D': // DC
                    if (chars[i + 1] == 'C') {
                        // DC
                        i++;
                        // check for preceding qty and subsequent 'in first/last stitch'
                        if (qty > 0) {
                            if (instruction.indexOf('DC in first stitch') > 0
                                        || instruction.indexOf('DC in last stitch') > 0) {
                                if (qty == 3) {
                                    symbol = Character.valueOf('w');
                                    printToken(symbol, qty);
                                } else if (qty == 2) {
                                    symbol = Character.valueOf('v');
                                    printToken(symbol, qty);
                                } else {
                                    System.out.printf('unexpected DC qty %d for instruction %s', qty, instruction);
                                }
                            } else {
                                System.out.printf('unexpected DC instruction %s with qty %d', instruction, qty);
                            }
                        } else {
                            // check for subsequent qty


                        }


                        else {
                            System.out.printf('unexpected DC instruction: %s', qty, instruction);
                        }


                    }
                    break;
                case 'S': // SC, SKqty
                    if (chars[i + 1] == 'C') {
                        // SC
                        // check for preceding qty
                        // check for succeeding qty

                    } else if (chars[i + 1] == 'K') {
                        // check for succeeding qty
                    }
                case 'C': //CHqty
                    if (chars[i + 1] == 'H') {
                        // check for succeeding qty
                    }
                }
            }

        }
        return "";

    }

    private String printToken(Character token, int qty) {
        if (token == null) {
            return "NULL";
        }
        final StringBuilder sb = new StringBuilder();
        for (int i = 0; i < qty; i++) {
            sb.append(token);
        }
        return sb.toString();
    }

    public String nextInstruction() {
        if (!hasNext()) {
            throw new RuntimeException("No more tokens available");
        } else if (position == 0) {
            position++;
            return "3DC in first stitch";
        }

        StringBuilder instruction = new StringBuilder();
        while (hasNext()) {
            char currentChar = input.charAt(position);
            if (currentChar == ',' || currentChar == '.') {
                position++;
                // if (token.instruction() == 0) {
                //     // Special character as a instruction itself
                //     instruction.append(currentChar);
                //     position++;
                // }
                break;
            } else if (currentChar == '{' || currentChar == '}' ) {
                // TODO something special, make sure it handles the 'X times' instruction that follows '}'
                position++;
                return "yrf";
            } else {
                instruction.append(currentChar);
                position++;
            }
        }
        return instruction.toString();
    }
}
