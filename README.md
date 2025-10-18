# Extended_Sed-msed_UNIX_Programming

New commands:
### f:  
Sets the flag that is used by t.

### F: 
Resets the flag that is used by t.

### Z: 
   Similar to how "z" zaps the Pattern Space (PS), "Z" will only zap up to  
   (and including) the first \n. If there's no \n, then zap all of the PS.

### W: 
   Consider that "p" prints the whole PS, but "P" prints just the first line.  
   Similarly, "w" writes the whole PS to a file, and so you'd think "W" would  
   print just one line to the file. Well, now that is what "W" does.

### C: 
   Changes the PS to the text the follows the C. But unlike "c", "C" does not  
   have any control flow side effect.

### D: 
   This modifies the behavior of traditional "D", because it now has consisent  
   behavior regardless of whether there is a "\n" in the PS: First, one line   
   is deleted from the PS (regardless of whether that line is offset by a "\n"  
   or is just the entire PS). Second, the sed program restarts - without  
   loading in the next line. In other words, D has been changed so that it  
   behaves consistently, and never behaves exactly like a d.  
  
### i,a,c,C,w,W,r:  
   all of these take the REST of the line as its argument. But not  
   in this new version. Now a ";" will end the argument.

### \$ -1,\$ -2, etc: 
   \$ is the last line of the file. But what about the 2nd-to-last?  
   It is \$ -1. 3rd-to-last? \$ -2. Etc.

### \$2,\$3,etc: 
   These now refer to passed-in arguments. This needs clarification,  
   in the footnotes below.

 -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -


### Footnote 1: 
            Why are the arguments \$2, \$3, ... defined, but not \$1? And why  
            does the msed template file say that \$1 gets store into t5?  
            The answer is that I have made a simplification to the assignment;  
            the first argument is always the program. Moreover, the input file  
            is never an argument. So, in our version of sed, it can ONLY be  
            run like this:  
                % cat infile | ./msed '<program>' flags or arguments.  

### Footnote 2: 
            What do the \$2, \$3, etc really mean? Well, consider this:  
            % echo ABC | ./msed 's/\$2/\$3/;/\$4/p' B b C -n  
            AbC  
            %  
            So, as you see, the argument is replaced by its value.  
            Now, there is a need to be careful with quoting. For example:  
            % echo ABC | ./msed 's/\$2/\\$3/;/\$4/p' B b C -n  
            A\$3C  
            %  

            The way this is accomplished is to not just check for the  
            expression \$[0-9]\{1,\}, but to also make sure that the character  
            before the "\$" is not a "\". But there is a little detail: what  
            if the \$ is the first character, so that you can't check the  
            character before it? The answer is: to make sure that there IS a  
            character before it -- which explians line 8 of the msed_template  
            file.  

### Footnote 3: 
            What is the strategy to handle backquoting?  
            We need to be careful about "\;" or "\\$3", etc -- and to not be   
            fooled by "\\;" or "\\\$3".  
            The solution is to:  
            - First, convert all the "\\" into "\h" (because "\h" is a special  
              character that will not occur in the infiles that test your   
              program with). But note: you need to use "\\\\" to get "\\".  
            - Second, convert "\;" to "\f" (another special character that  
              will not occur in the infiles).  
            - Third, put a "\n" before all remaining ";".  
              (Now, when you do this, there will never be two commands on one  
              line anymore -- but there might be a single command spread over  
              multiple lines (and this gets cleaned up by later parts of the  
              msed_template file).  
              

### Footnote 4: 
            What is the strategy to handle "F"?  
            The "F" command needs to reset the flag. This is accomplished  
            by do a branch. If you don't want to branch anywhere (you just  
            want to reset the flag), then branch to the place where you are  
            already (eg: ...;F;... => ...;bL;:L;...).  
            But there is a big issue: You can have more than one "F", and  
            they will need unique labels. And that is why Line 63 of the  
            msed_template file adds a counter to the label.  
