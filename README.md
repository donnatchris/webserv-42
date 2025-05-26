# PROJECT WEBSERV FOR 42
By chdonnat (Christophe Donnat from 42 Perpignan, France)

## AIM OF THE PROJECT:



### BONUS PART:



## SOME COMMANDS YOU CAN USE:

compile the program and suppress the .o files:

	make && make clean

execute the program

	./webserv <configuration_file>

 execute the program with valgrind

	make val

## ARCHITECTURE:
- minishell/ directory with the whole project
	- libft/ directory with the libft (+ get_next_line and ft_printf)
 	- dclst/ directory with functions and header for using doubly circular linked list
	- bonus/ directory with files for the bonus part
		- src/ directory containing the main files of the project
  			- builtins/ files for builtins commands
     			- env/ files with functions needed to interact with environment variables
        		- executor/ files to execute the command line
          		- lexer/ files for splitting the user input into tokens, store them in a chained list, check the syntax and create a binary tree
            		- signals/ files for handling signals
          		- text_transformer/ files for managing '$', '*' and '~'
		- utils/ directory for secondary files
		- include/ directory for headers
	- mandatory/ directory for the mandatory part (empty - everything is in the bonus directory)
- Makefile (with rules: make bonus clean fclean re)
- readme.md for quick explanation and main commands of the project
- valgrind.sup is a file containig a list of readline() leaks to suppress when executing valgring

## ABOUT MY PROJECT:
