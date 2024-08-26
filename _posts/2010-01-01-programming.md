---
date: 2019-05-16 23:48:05
layout: post
title: Programming a Malicious Backdoor and Command and Control (C2) Server in Python
subtitle: 
description:
image: https://i.imgur.com/DCgwIwy.jpeg
optimized_image: https://i.imgur.com/DCgwIwy.jpeg
category: blog
tags:
  - hacking
  - persistence
  - exploitation
  - post-exploitation
author: mindhackdiva
paginate: true
---
Command injection is a serious security vulnerability that enables attackers to manipulate a system's commands through a compromised application, leading to the unauthorized execution of arbitrary operations on the host operating system.

This form of attack capitalizes on the improper handling of user input by applications, thereby allowing malicious individuals to insert and execute commands within the system shell. Through this breach, and as an ethical hacker, you can leverage vulnerable entry points such as data input forms, cookies, and HTTP headers to infiltrate a system undetected. Once successfully infiltrated, the injected commands are executed within the environment under the guise of the compromised application's privileges, allowing you to potentially wreak havoc on the system.

This type of attack primarily thrives on the laxity in input validation protocols implemented by applications as the absence of stringent checks on user-supplied data opens the door for exploitation. When you take advantage of these types of weaknesses, it enables to you deploy carefully crafted commands to carry out, what some might call, _questionable_ tasks while concealing those tasks within legitimate application processes allowing the code to blend in with legitimate coding processes already in place. Ultimately, the prevalence of command injection attacks highlights the importance of performing validation and sanitization of user-inputs effectively in all distributed and deployed software applications to defend against these types of unauthorized systems access attacks and thwarting potential compromises at the same time.

Command injection attacks differ from that of a code injection attack as the key distinction lies in the fact that command injection is more about exploiting the application's inherent capabilities to execute commands, rather than introducing new code. By exploiting the application's command execution functionalities, you can effectively bypass security measures to gain unauthorized access to systems. It is important to point out that this method of attack can be particularly dangerous as it allows you to perform various types of system-level operations with the privileges of the application; therefore, posing a security challenge and threat to a system's integrity and confidentiality, and possibly availability.

Therefore, while both code injection and command injection are both serious security concerns, it is vital that as an ethical hacker and defender, you understand their variances and the nuances between the two distinct attacks. Understanding these key differences will allow you to better fortify against unauthorized intrusion and manipulation attempts these exploits aim to enact against any application if proper secure coding methods are not implemented or enforced for prevention.

<h2><strong>Example 1:</strong></h2>

The code provided below serves as a wrapper for the UNIX command `cat,` specifically designed to display the contents of a file on the standard output. This code is deemed injectable due to its vulnerability to potential command injections, which can lead to security issues. By including the necessary header files like `stdio.h` and `unistd.h` the program sets up the environment for executing system calls and handling input/output operations.


```js
#include <stdio.h>
#include <unistd.h>

int main(int argc, char **argv) {
  char cat [] = " cat ";
  char *command;
  size_t commandLength;

  commandLength = strlen(cat) + strlen(argv[1]) + 1;
  command = (char *) malloc(commandLength);
  strncpy(command, cat, commandLength);
  strncat(command, argv[1], (commandLength - strlen(cat)) );

  system(command);
  return (0);
}
```

Within the main function, the code initializes variables like `cat[]`, `command`, and `commandLength` to construct the final command that would be executed. The command string is created by concatenating the `cat` command with the file name provided as an argument (`argv[1]`). To ensure that the entire command string fits within the allocated memory space, the code calculates the required length and dynamically allocates memory using `malloc`. Subsequently, the command string is formed by copying the `cat` command and then appending the file name.

This implementation may pose a serious risk of untrusted input is not properly sanitized, as it allows the execution of arbitrary commands alongside the `cat` command; therefore, caution must be exercised to prevent potential malicious activities through command injection attacks. By executing the system function with the constructed command, the code effectively prints the contents of the specified file to the standard output. Finally, the function returns `0` to indicate successful completion of the programs execution.

When `catWrapper` is utilized as intended, it will simply display the requested file's contents, as illustrated by the command: `$ ./catWrapper Story.txt`. Here, we find our story picking up where we last left our heroes, a seamless continuation of their journeys. 


```js
$ ./catWrapper Story.txt
We last left our heroes at...
```

However, intriguingly enough, if a semicolon is appended followed by another command like l`s`, `catWrapper` will go ahead and execute this combined command without raising any objections, as demonstrated by: `$ ./catWrapper "Story.txt; ls.` As a result, not only do we get to read where our heroes last left off, but we also receive a listing of files residing in the same directory, providing us with additional information in one fail swoop.


```js
$ ./catWrapper "Story.txt; ls"
We last left our heroes at...
Story.txt             doubFree.c
nullpointer.c
unstosig.c            www*            a.out*
format.c              strlen.c
useFree*
catWrapper*           misnull.c
strlength.c           useFree.c
commandinjection.c    nodefault.c		  trunc.c
writeWhatWhere.c
```

This nuanced behavior also unveils another security concern on top of command injection. If `catWrapper` were assigned a privilege level above that of a standard user, there exists a potential vulnerability where unauthorized users could exploit this to run any command with the enhanced privileges, giving them broader access and control over the system than they should have. This poses a significant security risk as it opens up the possibility of running arbitrary commands with elevated privileges, and, in turn, can lead to misuse or unintended-consequences within the system's operating environment. All in all, this simple yet powerful tool must be carefully monitored and configured to prevent any such security breaches.

<h2><strong>Example 2:</strong></h2>

The simple program described above follows a specific functionality where it accepts a filename as a command-line argument and the proceeds to exhibit the contents of the file back to the user. Its unique feature is the installation as a `setuid` root program, a deliberate action taken to position it as an educational tool for aspiring hackers and sysadmins alike. This installation choice empowers you a with an opportunity to explore and learn crucial system files without the risk of inadvertently modifying them or causing any harm to the system itself.


```js
int main(char* argc, char** argv) {
  char cmd[CMD_MAX] = "/usr/bin/cat ";
  strcat(cmd, argv[1]);
  system(cmd);
}
```

The function within the program, as shown in the above code, essentially generates a command using `/usr/bin/cat` followed by the specified file name provided by the user as an argument. This constructed command is then executed via the `system()` call. It is important to note that this entire process, including the invocation of the `system()` function, operates under the root privileges granted to the program.

While the program's default behavior performs as intended when handling standard file names, this, again, introduces another potential vulnerability when you attempt to exploit the program's functionalities. For instance, if you cleverly craft and insert a string like `;rm -rf /`, the `system()` call encounters unexpected behaviors, thus failing to execute the `cat` command due to the manipulated arguments. Consequently, this failure triggers a cascading effect, leading to the inadvertent recursive deletion of the root partition's contents, posing a significant risk to the overall system integrity and potentially causing substantial damage if left unchecked.

<h2><strong>Example 3:</strong></h2>

In this vulnerable syntactical code shown below, a privileged program leverages the environment variable `$APPHOME` to ascertain the precise location of the application's installation directory. Consequently, it then proceeds to initiate an initialization script within that identified directory:


```js
...
char* home=getenv("APPHOME);
char* cmd=(char*)malloc(strlen(home)+strlen(INITCMD));
if (cmd) {
  strcpy(cmd,home);
  strcat(cmd, INITCMD);
  execl(cmd,NULL);
}
...
```

In the mentioned context, the flawed code provides a doorway of sorts for you to carry out commands using the elevated privileges granted to the application. To elaborate further on the security vulnerability exhibited here, malicious entities can manipulate the `$APPHOME` environment variable to disclose an alternative pathway containing a corrupted version of the `INITCMD` script. This susceptibility arises form the absence of input validation mechanisms within the program, ultimately paving the way for threat actors to deceive the application into executing harmful scripts.

By exploiting the environment variable, you gain the ability to dictate the specific commands executed by the program, thereby highlighting the direct impact of the environment's influence in this demonstrated scenario. Moving forward, it is crucial to explore the potential consequences that may unfold when you, or an attacker chooses to manipulate the interpretation of the command sequence, thereby intensifying the risks associated with unauthorized code executions.

<h2><strong>Example 4:</strong></h2>

The syntax below originates from a web-based CGI utility tailored for users to modify their passwords securely. In the context of the Network Information Service (NIS) environment, upgrading passwords involves the execution of `make` within the `/var/yp` directory. It's important to note that the utility operates with escalated privileges, specifically as `setuid` root, to modify password records quickly and efficiently.

To initiate the `make` command, the program employs the following syntax:

`system("cd /var/yp && make &> /dev/null");`

A notable distinction in this instance is the hardcoded command, which mitigates an attacker's ability to manipulate the argument passed to the `system()`. Nevertheless, the absence of an absolute path for `make` and the program's failure to sanitize environment variables before executing the command introduce a potential security vulnerability. Exploiting this gap, an attacker could manipulate the `$PATH` variable to point towards a malicious `make` binary, thus ingeniously compromising the CGI script's execution from a shell prompt. Once the attacker's version of `make` gains control, it operates with root privileges due to the utility's `setuid` root installation, magnifying the criticality of this loophole in the security architecture. Consequently, this intricate chain of events underscores the need of stringent input validations and comprehensive environment scrutiny to fortify the integrity of the system against such malicious exploits.

The environment in which system commands are executed within programs wields significant influence on their behaviors. When utilizing functions like `system()` and `exec()`, the program that initiates these calls also dictates the environment in which they will run. This aspect, as you may have guessed by now, opens up potential vulnerabilities for exploitation and manipulation of the outcome these commands will produce.

It is commonly misconstrued that Java's Runtime.exec function mirrors C's system function, but this is not accurate. Although both functions serve the purpose of launching new programs or processes, they operate differently. In the case of C's system function, arguments are passed to the shell (`/bin/sh`) for parsing. In contrast, `Runtime.exe` parses the input string into an array of words and then executes the first word as a command, and with the subsequent words serving as parameters. Notably, `Runtime.exec` does not involve invoking the shell at any point during its execution. This distinction is crucial as it mitigates many vulnerabilities typically associated with shell usage, such as command chaining using `&`, `&&`, `|`, `||,` input/output redirection, and more. Instead, these potentially harmful actions would be treated simply as parameters passed to the initial command, likely resulting in syntax errors or being discarded as invalid inputs in the process.

<h2><strong>Example 5:</strong></h2>

The provided code demonstrate a vulnerability to OS command injection on Unix/Linux systems. Within the C program, if the number of arguments is not equal to 2, it outputs an error message. To set up the command for execution, the program concatenates the input argument with a predefined string. Upon invoking the `system` command, it can execute the concatenated command.

In the scenario of this being a `setuid` binary, an attacker could exploit the vulnerability by entering a malicious command separated by a semi-colon. 

Consider the example: `"ls; cat /etc/shadow"`. Because Unix shells interpret commands separated by semicolons as distinct commands, the attacker could chain multiple malicious commands. This risky practice enables the attacker to execute arbitrary system commands with the privileges of the vulnerable program, illustrating the severity of the command injection threat.


```js
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
     char command[256];

     if(argc != 2) {
          printf("Error: Please enter a program to time!\n");
          return -1;
     }

     memset(&command, 0, sizeof(command));

     strcat(command, "time ./");
     strcat(command, argv[1]);

     system(command);
     return 0;
}
```

By strategically crafting input, attackers can bypass intended security measures and gain unauthorized access or perform malicious actions. In this case, the lack of input validation and sanitization opens the door for potentially devastating consequences. It highlights the critical importance of input validation, proper handling of user input, and secure coding practices to mitigate the risks posed by command injection vulnerabilities. Prompt attention to these vulnerabilities can safeguard systems and protect sensitive information from exploitation.

<h2><strong>Java</strong></h2>

There are numerous online resources that incorrectly claim Java’s `Runtime.exec` and C’s system function are functionally identical; however, in reality, there are distinct differences between the two. While both methods allow for the invocation of a new program or process, the way they handle arguments differs significantly. In C, the system function passes its arguments to the shell (`/bin/sh`) for parsing, whereas Runtime.exec in Java attempts to split the input string into an array of words. Subsequently, it executes the first word in the array using the remaining words as parameters. It is crucial to note that `Runtime.exec` does not involve invoking the shell at any point during execution.

The critical disparity lies in how the shell's functionalities are handled. In C, the shell possesses capabilities that could potentially lead to misuse, such as command chaining using symbols like `&`, `&&`, `|`, `||`, and redirecting input and output; however, with Runtime.exec, these features are not directly accessible. Instead, any attempt to exploit such functionalities through the shell would result in those commands being treated as ordinary parameters of the initial command. This could lead to syntax errors or result in the commands being discarded as invalid parameters, highlighting the fundamental divergence in behavior between the two methods.

<h2><strong>Example 6:</strong></h2>

The following PHP code is vulnerable to a command injection attack:

```js
<?php
print("Please specify the name of the file to delete");
print("<p>");
$file=$_GET['filename'];
system("rm $file");
?>
```

The following request and response is an example of a successful attack:

```js
Request http://127.0.0.1/delete.php?filename=bob.txt;id
Response
Please specify the name of the file to delete:
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

<h2><strong>Sanitizing Input:</strong></h2>

```js
Replace or Ban arguments with “;”
Other shell escapes available
Example:
- &&
- |
- ...
```

<h2><strong>Related Controls</strong></h2>

In a perfect world, a developer should understand how to utilize existing APIs specifically designed for their chosen programming language to streamline the development process and ensure compatibility. For instance (in the context of Java), rather than resorting to using `Runtime.exec()` to execute a `mail` command, it is advisable to leverage the robust Java API provided within the `javax.mail.*` package. This approach not only enhances code readability and maintainability but also minimizes potential vulnerabilities that may arise from invoking system-level commands.

In cases where a suitable API is not readily available, developers need to learn to adopt proactive measures to sanitize all input data effectively, thereby mitigating the risk of malicious attacks and unauthorized access. By incorporating and implementing a positive security policy or procedural model into your client's overall security Policies, Procedures, and Guidelines (PPGs) strategies, developers can establish a secure foundation that prioritizes the acceptance of known safe characters over the rejection of potentially harmful input. Generally, defining the set of permissible characters proves to be a more straightforward task than enumerating all the forbidden characters, as it allows for clearer validation logic and facilitates accurate data filtering mechanisms.

Therefore, adhering to these best practices and security standards not only bolsters the overall robustness of the software but also promotes a proactive approach to handling input validation and security considerations effectively. By incorporating these principles into the development workflow, you can show and train them on how to better and best safeguard their applications against common security threats, maintaining high levels of integrity in handling user inputs from both attacking and defending perspectives.

image: https://github.com/user-attachments/assets/042bbaa3-19fc-4669-b549-44c794c728ab


## Inline HTML elements

HTML defines a long list of available inline tags, a complete list of which can be found on the [Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/HTML/Element).

* **To bold text**, use `<strong>`.
* *To italicize text*, use `<em>`.
* Abbreviations, like <abbr title="HyperText Markup Langage">HTML</abbr> should use `<abbr>`, with an optional `title` attribute for the full phrase.
* Citations, like <cite>&mdash; Thomas A. Anderson</cite>, should use `<cite>`.
* <del>Deleted</del> text should use `<del>` and <ins>inserted</ins> text should use `<ins>`.
* Superscript <sup>text</sup> uses `<sup>` and subscript <sub>text</sub> uses `<sub>`.

Most of these elements are styled by browsers with few modifications on our part.

# Heading 1

## Heading 2

### Heading 3

#### Heading 4

Vivamus sagittis lacus vel augue rutrum faucibus dolor auctor. Duis mollis, est non commodo luctus, nisi erat porttitor ligula, eget lacinia odio sem nec elit. Morbi leo risus, porta ac consectetur ac, vestibulum at eros.

--page-break--

## Code

Cum sociis natoque penatibus et magnis dis `code element` montes, nascetur ridiculus mus.

```js
// Example can be run directly in your JavaScript console

// Create a function that takes two arguments and returns the sum of those arguments
var adder = new Function("a", "b", "return a + b");

// Call the function
adder(2, 6);
// > 8
```

Aenean lacinia bibendum nulla sed consectetur. Etiam porta sem malesuada magna mollis euismod. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa.

## Lists

Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Aenean lacinia bibendum nulla sed consectetur. Etiam porta sem malesuada magna mollis euismod. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa justo sit amet risus.

* Praesent commodo cursus magna, vel scelerisque nisl consectetur et.
* Donec id elit non mi porta gravida at eget metus.
* Nulla vitae elit libero, a pharetra augue.

Donec ullamcorper nulla non metus auctor fringilla. Nulla vitae elit libero, a pharetra augue.

1. Vestibulum id ligula porta felis euismod semper.
2. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus.
3. Maecenas sed diam eget risus varius blandit sit amet non magna.

Cras mattis consectetur purus sit amet fermentum. Sed posuere consectetur est at lobortis.

Integer posuere erat a ante venenatis dapibus posuere velit aliquet. Morbi leo risus, porta ac consectetur ac, vestibulum at eros. Nullam quis risus eget urna mollis ornare vel eu leo.

## Images

Quisque consequat sapien eget quam rhoncus, sit amet laoreet diam tempus. Aliquam aliquam metus erat, a pulvinar turpis suscipit at.

![placeholder](https://placehold.it/800x400 "Large example image") ![placeholder](https://placehold.it/400x200 "Medium example image") ![placeholder](https://placehold.it/200x200 "Small example image")

## Tables

Aenean lacinia bibendum nulla sed consectetur. Lorem ipsum dolor sit amet, consectetur adipiscing elit.

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Upvotes</th>
      <th>Downvotes</th>
    </tr>
  </thead>
  <tfoot>
    <tr>
      <td>Totals</td>
      <td>21</td>
      <td>23</td>
    </tr>
  </tfoot>
  <tbody>
    <tr>
      <td>Alice</td>
      <td>10</td>
      <td>11</td>
    </tr>
    <tr>
      <td>Bob</td>
      <td>4</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Charlie</td>
      <td>7</td>
      <td>9</td>
    </tr>
  </tbody>
</table>

Nullam id dolor id nibh ultricies vehicula ut id elit. Sed posuere consectetur est at lobortis. Nullam quis risus eget urna mollis ornare vel eu leo.
