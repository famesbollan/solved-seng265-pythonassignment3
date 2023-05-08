Download Link: https://assignmentchef.com/product/solved-seng265-python_assignment3
<br>
<ul>

 <li>Use the Python programming language to write your implementation of basic OLAP (Online Analytical Processing) queries from first principles, in a file named “OLAP.py”</li>

 <li>Test your code against the provided test data, and other test data you create, to ensure it’s working as expected. The instructor will also be testing your program against test data that is not provided to you. Think creatively about how to “break” and strengthen your code!</li>

</ul>

In this assignment, we will implement OLAP queries from first principles. There are numerous libraries that perform these functions, however, learning how to do this from first principles will help to illustrate a few key benefits:

<ul>

 <li>We can better understand the library implementations and their usage</li>

 <li>We can demystify some of what goes into library development</li>

 <li>We can get hands-on with some of Python’s strong abilities in processing big data</li>

</ul>




To this extent, we will write code that will calculate the <strong>min​</strong>​ , <strong>max​</strong>​     , <strong>mean​</strong>​  , <strong>sum​</strong>​     and <strong>count​</strong>​         of numeric columns and the <strong>group-by​</strong>​      along with <strong>top-k​</strong>​            for categorical columns.




We’ll take as our input a set of time-stamped events (these will be supplied in a CSV file) – and we’ll generate some summary calculations for the whole file. At present, we seem to find ourselves in a “golden era” of data science, so perhaps some of these techniques may prove useful to you.




<h1>Background Information</h1>




<h2>Data Science</h2>




A data scientist is someone who applies a combination of insights from computer science and statistics to solve problems relating to information (a.k.a data) — in particular, the extraction of insights from complex data. Data scientists use numerous libraries, frameworks, modules and toolkits to efficiently implement the most common data science algorithms and techniques.

When you’re approaching data science with the Python programming language, some of the useful tools you’re likely to encounter include <strong>NumPy​</strong>​       and <strong>pandas</strong>​      <a href="#_ftn1" name="_ftnref1"><strong><sup>[1]</sup></strong></a> – they’re well-acknowledged for the aid they provide in data science tasks, and abstract away much of the complexity in their underlying implementations. However, by abstracting away said complexity, on the surface that definitely supports bringing more people into the world of data science, but it also shields us from acquiring deeper understanding.




If you are trying to implement your own versions of some well-known functions, you may not end up with the “optimal” implementation (e.g. your implementation may not be able to handle an “astounding level of data”), but you’ll definitely advance your understanding as a practitioner… and who knows, having the knowledge on hand could potentially help during a job interview.




In this assignment, we’ll ​<em>implement some well-known functions from first principles</em>​. The functions we will use in this assignment will be defined in the following sections.




<h2>Additional Notes</h2>




<ul>

 <li>It’s possible to produce all of the output required for this assignment in ​<em>a single traversal of the input file</em>​, i.e. reading each input record only once. This may not sound so important when you’re dealing with a few thousand records, but if the big data is really big, or even infinite (in the case of streaming data) it becomes really important to be efficient.</li>

 <li>A ​<strong><em>numeric field</em></strong>​ is one that contains values intended to be interpreted as numbers. They may be integers, floating-point numbers, missing values, or a combination</li>

 <li>A ​<strong><em>categorical field</em></strong>​ is one that contains values ​<em>not</em>​ intended to be interpreted as numbers, even though they sometimes consist of nothing but digits (e.g. UNIX timestamps; auto-incrementing primary keys in a database table)</li>

 <li><em>NaN</em>​ means “Not a Number” — a valid floating point value that is not zero and not infinite, but also… not a number</li>

 <li>A ​<em>CSV file</em>​ is a plain-text file that has “comma-separated values” – while a CSV file can be read by any application, it consists of contents that are well-supported through spreadsheet applications like Microsoft Excel ● Example CSV file:

  <ul>

   <li>csv</li>

  </ul></li>

</ul>

■ timestamp,colour,sound

■ 2019-11-03T03:45:56.000Z,red,moo

■ 2019-11-04T01:15:78.033Z,green,oink

<ul>

 <li>Some CSV dialects use null​​ or “”​​     to represent missing values; others just repeat the delimiter, e.g. in the following example, Bob does not have a pet:</li>

</ul>

name,pet_type,age Alice,Angora Rabbit,11

Bob,,13

Clara,Domestic Shorthair Cat,15

<ul>

 <li>The first line of the CSV file will be a standard ​<em>CSV header line</em>​, in which the <strong>field names​</strong>​ are listed. Each subsequent line will contain values in those fields. Lines are separated by line feeds (ASCII code #10, aka Unix “new line” characters, aka 
 — CSV files from Windows systems will often use two characters between lines, CRLF (Carriage Return, Line Feed) — you can use dos2unix to strip out the carriage returns if you encounter one. (Reminder: your program <u>MUST</u><u>​       </u> run in the Lab environment)<u>​    </u></li>

 <li><strong>Aggregate functions</strong> are mathematical functions that take a (potentially very large) number of input values, and produce a​ single output value. (As an aside to the current assignment: In SQL<a href="#_ftn2" name="_ftnref2"><sup>[2]</sup></a>, aggregate functions can appear as “projections”, e.g. SELECT min(age), max(age) FROM KidsWithPets)</li>

 <li>Commonly used aggregate functions have different memory requirements:

  <ul>

   <li>constant space (e.g. sum, min, max, mean)</li>

  </ul></li>

</ul>

○    O(n) in the number of input values (e.g. accurate median)

○    O(n) in the number of distinct values observed (e.g. top-k, count distinct)




<h1>Command line specification (your program <u>must</u>​ run in the following form)​</h1>




<em>python OLAP.py –input &lt;input-file-name&gt; [aggregate args] [–group-by &lt;fieldname&gt;] </em>




<h2>Arguments (which may occur in any order)</h2>




<strong>–input &lt;input-file-name&gt;</strong>​ (e.g. –input input.csv) name of the input file to be processed. The input file must contain a header line.




<strong>aggregate args </strong>​(​see [Aggregate Functions] below for a list of supported functions):

specifies the aggregate functions to calculate on the file. Any number of aggregates (zero or more) can be specified. If no aggregates are specified, default to –count. If one of the aggregate function arguments references a nonexistent field, program exits with error code 8​ ​, prints on stderr​​              , Error: &lt;input_file&gt;:no field with name ‘&lt;missing_field&gt;’ found​




<strong>–group-by &lt;categorical-field-name&gt; </strong>program computes the requested aggregates independently for each distinct value in &lt;categorical-field-name&gt;. If there are more

than 1000 distinct values in categorical-field-name, the program will use the first 1000 values found, and message on stderr to indicate partial output, i.e. Error: &lt;inputfile&gt;: &lt;categorical_field&gt; has been capped at 1000 values​   <strong>Aggregate Functions </strong>




<table width="880">

 <tbody>

  <tr>

   <td width="241"><strong>argument </strong></td>

   <td width="208"><strong>description </strong></td>

   <td width="431"><strong>output format </strong></td>

  </tr>

  <tr>

   <td width="241">–top &lt;k&gt; &lt;categorical-field-name&gt;</td>

   <td width="208">compute the top k ​​         most common values of categorical-field-name</td>

   <td width="431">a string listing the top values with their counts, in descending order, e.g. “red: 456, green: 345, blue: 234”If the values contain double quote or newline characters, they should be escaped using the usual printf syntax, e.g. “
” for newlines, ” for double quotes</td>

  </tr>

  <tr>

   <td width="241">–min &lt;numeric-field-name&gt;</td>

   <td width="208">compute the minimum​​ value of numeric-field-name</td>

   <td width="431">a floating point number, or “NaN” if there were no numeric values in the column</td>

  </tr>

  <tr>

   <td width="241">–max &lt;numeric-field-name&gt;</td>

   <td width="208">compute the maximum​​               value of numeric-field-name</td>

   <td width="431">a floating point number, or “NaN” if there were no numeric values in the column</td>

  </tr>

  <tr>

   <td width="241">–mean &lt;numeric-field-name&gt;</td>

   <td width="208">compute the mean​​        (average) of numeric-field-name</td>

   <td width="431">a floating point number, or “NaN” if there were no numeric values in the column</td>

  </tr>

  <tr>

   <td width="241">–sum &lt;numeric-field-name&gt;</td>

   <td width="208">compute the sum​​           of numeric-field-name</td>

   <td width="431">a floating point number, or “NaN” if there were no numeric values in the column</td>

  </tr>

  <tr>

   <td width="241">–count</td>

   <td width="208">count​ the number of records</td>

   <td width="431">an integer, or zero if there were no records</td>

  </tr>

 </tbody>

</table>




<strong>Group By </strong>

Add optional named argument for a group-by​

–group-by categorical-field-name




If a group-by​​     field is specified:

<ul>

 <li>If it’s a valid categorical field, produce an output CSV file with one row of output per distinct value in that field, with: – the value of the group-by field in the first column</li>

 <li>the values of any computed aggregates in subsequent columns, in the order they were specified at the command line (or just a count column, if no aggregates were requested at the command line)</li>

 <li>If input CSV file does not contain a field with a name matching –​ group-by​ argument, exit with error code 9​ ​, and print on stderr​​ , ​Error: &lt;input_file&gt;:no group-by argument with name ‘&lt;group-by_argument&gt;’ found3</li>

 <li>If the field is not a categorical field, or has very high cardinality , or is sparse, use your judgment on what to return.</li>

</ul>

<sup>3</sup> Cardinality is a common idea in big data – high cardinality refers to situations in which data has numerous unique values (low cardinality​

—&gt; data has few unique values); when we have millions of distinct values, dynamic memory becomes crucial

If no –​ group-by​ argument is specified, then produce the requested aggregates for the whole file in a single-line (​<u>plus header</u>​) CSV output file with no extra column for a group-by field.

<h1><strong>Group-By Example Output </strong></h1>

The query is “–​ group-by colour –count –min foo​”

The results indicate:

<ul>

 <li>There are four distinct values in the colour​​ field</li>

 <li>The number of records with colour == red​​ is 123, etc.</li>

 <li>The minimum value of foo​​ for all records with colour == red​​           is 3, etc.</li>

</ul>

<table width="879">

 <tbody>

  <tr>

   <td width="293"><strong>colour </strong></td>

   <td width="293"><strong>count </strong></td>

   <td width="293"><strong>min_foo </strong></td>

  </tr>

  <tr>

   <td width="293">red</td>

   <td width="293">123</td>

   <td width="293">3</td>

  </tr>

  <tr>

   <td width="293">green</td>

   <td width="293">234</td>

   <td width="293">4</td>

  </tr>

  <tr>

   <td width="293">blue</td>

   <td width="293">345</td>

   <td width="293">5</td>

  </tr>

  <tr>

   <td width="293">violet</td>

   <td width="293">456</td>

   <td width="293">6</td>

  </tr>

 </tbody>

</table>

<strong> </strong>Requirements

<strong>Normal Operation </strong>

<ol>

 <li>Check that the CSV file provided in the first command-line argument exists and is readable</li>

 <li>Validate the command line arguments</li>

 <li>Check that the requested computations are valid, given the fields declared in the input file’s header</li>

 <li>Read the file, during which you should:

  <ol>

   <li>keep track of which line you’re reading, so you can provide better error messages.</li>

   <li>compute the aggregates requested (or just –count if none were requested)</li>

   <li>split the output by the group-by field if requested, or produce one row if no group-by ​​ was requested.</li>

  </ol></li>

</ol>

<ol start="6">

 <li>Output should be in CSV format on stdout

  <ol>

   <li>you can use ​`​python OLAP.py –count ​<strong>&gt; somefile.csv​</strong>`​, for example, to capture your program’s output to a file so it may be ​<em>compared</em></li>

  </ol></li>

 <li>count, min, max​ and mean​​ aggregate results should be printed as numbers</li>

 <li>top-k​ aggregate results should be printed in one column, as strings with the following format, in descending order of count:</li>

</ol>

(value​<sub>1​</sub>: count​<sub>1</sub>)[<sub>​ </sub>, (value​<em><sub>n​</sub></em>: count​<em><sub>n​</sub></em>)…]

e.g.

blue: 456, green: 345, red: 234

<ol start="8">

 <li>Output fields should be named according to the aggregate function and original field name, separated by an underscore, e.g. “min_foo​​ ”, “avg_bar​​          ”, “top_colour​​    ”</li>

 <li>The first row in the output CSV file ​<u>must be a header row</u>​, with each column in the header row listing named accordingly (see examples, which include header rows, to better understand the naming in header)</li>

</ol>

<strong>Error Conditions — </strong>​<strong><u>all errors must be displayed on </u></strong><strong><u>stderr</u></strong><u>​            </u><strong>​</strong>, each on its own line (this permits us to direct the successful output of a program to a file, but still prints error messages to the screen – this distinction is vitally important and will be graded heavily, up to 30%)

<ol>

 <li>If an aggregate is requested on a field that contains non-numeric values, skip that value, and print an error message to stderr (not stdout!) with the following format<a href="#_ftn3" name="_ftnref3"><sup>[3]</sup></a>:</li>

</ol>

Error: &lt;inputfile&gt;:&lt;lineNumber&gt;: can’t compute &lt;aggregate&gt; on non-numeric value ‘&lt;value&gt;​ ‘

e.g.

Error: input.csv:123: can’t compute mean on non-numeric value ‘asdf’

<ol start="2">

 <li>If more than 100 non-numeric values are found in any one aggregate column, exit with error code 7, and print (on stderr) a message, Error: &lt;input_file&gt;:more than 100 non-numeric values found in aggregate column​ ‘&lt;aggregate_field&gt;’</li>

 <li>If a top-k​​ aggregate is requested on a field with more than 20 distinct values:

  <ul>

   <li>print a message on stderr​​ (not stdout​​        !) that the field has been capped at 20 distinct values, i.e. Error:​</li>

  </ul></li>

</ol>

&lt;inputfile&gt;: &lt;aggregate_field&gt; has been capped at 20 distinct values

<ul>

 <li>add “_capped” to the field name, e.g. “top_colour_capped”</li>

</ul>

<ol start="4">

 <li>If a group-by​​ field has more than 20 distinct values:

  <ul>

   <li>print a message on stderr​​ (not stdout​​        !) that the field has been capped at 20 distinct values, i.e. Error:​</li>

  </ul></li>

</ol>

&lt;inputfile&gt;: &lt;group_by_field&gt; has been capped at 20 distinct values

<ul>

 <li>assign the overflow records to an overflow row named “_OTHER” so the aggregates from the overflow records still get counted. The _OTHER row should be printed last.</li>

</ul>

<ol start="5">

 <li>Should you detect any other error that prevents the program from producing meaningful output, exit with error code 6 and a</li>

</ol>

custom error message of the form: is replaced with your custom message indicating what went wrong in an appropriate level of detail.Error: &lt;input_file&gt;:&lt;meaningful_error_message&gt;​​                                                                                 , where the tagged message

Example to guide your effort

“Almost the Kitchen sink” example (limited to only four companies to save space in this document, actual output would list all)

python OLAP.py –input input.csv –group-by ticker –count –min open –max open –mean open –min high –max high –mean high –min low –max low –mean close –min close –max close –mean close




<table width="916">

 <tbody>

  <tr>

   <td width="60"><strong>ticker </strong></td>

   <td width="58"><strong>count </strong></td>

   <td width="84"><strong>min_open </strong></td>

   <td width="58"><strong>max_ open </strong></td>

   <td width="96"><strong>mean_open </strong></td>

   <td width="80"><strong>min_high </strong></td>

   <td width="66"><strong>max_hi gh </strong></td>

   <td width="69"><strong>mean_</strong><strong>high </strong></td>

   <td width="67"><strong>min_lo w </strong></td>

   <td width="53"><strong>max_</strong><strong>low </strong></td>

   <td width="56"><strong>mean</strong><strong>_low </strong></td>

   <td width="53"><strong>min_</strong><strong>close </strong></td>

   <td width="54"><strong>max_</strong><strong>close </strong></td>

   <td width="62"><strong>mean_</strong><strong>close </strong></td>

  </tr>

  <tr>

   <td width="60">aapl</td>

   <td width="58">8364</td>

   <td width="84">0.23305</td>

   <td width="58">175.11</td>

   <td width="96">22.2843502</td>

   <td width="80">0.23564</td>

   <td width="66">175.61</td>

   <td width="69">22.4958666</td>

   <td width="67">0.23051</td>

   <td width="53">174.27</td>

   <td width="56">22.281018</td>

   <td width="53">0.23051</td>

   <td width="54">175.61</td>

   <td width="62">22.281018</td>

  </tr>

  <tr>

   <td width="60">adbe</td>

   <td width="58">7876</td>

   <td width="84">0.2</td>

   <td width="58">182.88</td>

   <td width="96">26.5608488</td>

   <td width="80">0.2</td>

   <td width="66">184.44</td>

   <td width="69">26.9107583</td>

   <td width="67">0.2</td>

   <td width="53">181.06</td>

   <td width="56">26.5774755</td>

   <td width="53">0.2</td>

   <td width="54">184.06</td>

   <td width="62">26.5774755</td>

  </tr>

  <tr>

   <td width="60">amzn</td>

   <td width="58">5153</td>

   <td width="84">1.41</td>

   <td width="58">1126.1</td>

   <td width="96">181.747357</td>

   <td width="80">1.45</td>

   <td width="66">1135.54</td>

   <td width="69">183.880652</td>

   <td width="67">1.31</td>

   <td width="53">1124.06</td>

   <td width="56">181.769343</td>

   <td width="53">1.4</td>

   <td width="54">1132.88</td>

   <td width="62">181.769343</td>

  </tr>

  <tr>

   <td width="60">axp</td>

   <td width="58">11556</td>

   <td width="84">0.49838</td>

   <td width="58">96.42</td>

   <td width="96">22.7530326</td>

   <td width="80">0.5074</td>

   <td width="66">96.73</td>

   <td width="69">23.0014514</td>

   <td width="67">0.48981</td>

   <td width="53">96.29</td>

   <td width="56">22.7552431</td>

   <td width="53">0.48981</td>

   <td width="54">96.43</td>

   <td width="62">22.7552431</td>

  </tr>

 </tbody>

</table>

…

<h2>Provided Materials</h2>

We have created a git repository for you – it contains a starter file, OLAP.py, your data set, a limited test suite, and these assignment instructions. There are also is also a hints section in this document.

OLAP.py

The starter file for this assignment, and where you are instructed to place your implementation.

Data Set

We have supplied a single test data file in your root directory. The dataset for this assignment (input.csv​​              ) contains ticker info for twenty large companies (across three sectors): Apple (aapl​​              ), Adobe (adbe​​  ), Amazon (amzn​​            ), American Express (axp​​          ), Ali Baba (baba​​            ), Salesforce (crm​​          ), Cisco (csco​​      ), Disney (dis​​    ), Facebook (fb​​               ), Google (​googl​              ), IBM (ibm​​        ), Intel (intc​​        ), Mastercard (ma​​            ), Microsoft (msft​​           ), Netflix (nflx​​   ), Nike (nke​​       ), Nvidia (nvda​​ ), Oracle (orcl​​   ), Starbucks (sbux​​           ) and Visa (​v​ ).

The data consists of nine columns, Sector​​     , Ticker​​ , Date, Open, High, Low, Close, Volume, and OpenInt​​      . The full dataset may be found <u>​</u><a href="https://www.kaggle.com/borismarjanovic/price-volume-data-for-all-us-stocks-etfs/data">here</a><a href="#_ftn4" name="_ftnref4"><sup>[4]</sup></a>. You can create your own .csv files, with your own data; once you have your implementation completing with the provided data, please feel free to test it with larger and different data sets as you see fit.

Libraries​ (permitted usage list, if library is not on this list then it may not be used)

+ csv​ (for parsing the input file, and for creating the output file as well)

+ argparse​ (for parsing parameters)

+ os

+ sys

Test Suite

We will supply a limited suite of test cases that you can use to evaluate your own program, and get a feel for the types of inputs you can expect. The test suite has the same structure as previous assignments – a set of directories, with one test case provided per directory. The tests shall run similarly to previous assignments. We also provide a file, command.log, to illustrate test case creation.

Hints (we may add more to a course announcement page at a later time)

–             while calculating the aggregate of a set of data, watch for situations in which there is no data to calculate

Process

To get started, the assignment repository has already been set up in your person GitLab space, listed as /assignment3.git. We set this up in our local machine space with<strong> git clone​</strong>​ (all one line, with &lt;NETLINK_ID&gt; replaced with your personal Netlink identifier):

<strong>git clone https://gitlab.csc.uvic.ca/courses/2019091/SENG265/assignments/&lt;NETLINK_ID&gt;/assignment3.git  </strong>

To ensure you and the teaching team can see your code, git push​​ to the same personal repository you cloned from GitLab!

Use your favourite text editor to write your program. Once you’re able to compile your program, test it by running it with the <strong>input.csv​ </strong>​files in the provided test suite, and any other files you can think of. Be creative; try to predict what kinds of inputs will break your code! Try random files lying around on your computer or the internet to see what happens!

When you’ve made changes to the file that you want to keep,<strong> git commit </strong>​your changes to your local copy of the repository. Git will save the history of past commits. When you want to upload your changes to your repository, use<strong> git push​</strong>​ .Deliverables

<ul>

 <li>Write a Python program in a file named OLAP.py (case sensitive), <u>​placed in the root of your assignment3 repository</u>​, that implements the above requirements</li>

 <li>The program must run successfully using ​<strong>python</strong>​ (within the lab environment, which runs Python 3.7) – We will run the script (possibly with aggregate args (and other parameters) in many different orders) with:</li>

 <li><strong>$ python OLAP.py –input input.csv [aggregate args] [–group-by &lt;fieldname&gt;]</strong></li>

 <li>where input_file.csv are the supplied ticker data</li>

 <li>You may only use use python3 libraries/modules specified in this assignment, if not specified, you may not use it library</li>

 <li>You may assume that all test files will be in the same CSV format provided with the assignment</li>

 <li>The assignment will be marked based on the last version of your program pushed before the deadline</li>

</ul>

<a href="#_ftnref1" name="_ftn1">[1]</a> these libraries, along with others that address this problem space, are also ​       ​<strong><u>strictly prohibited</u></strong><u>​</u> for use on this assignment

<a href="#_ftnref2" name="_ftn2">[2]</a> SQL a.k.a. the “structured query language”, highly popular in the management of relational database management systems (RDBMS)​

<a href="#_ftnref3" name="_ftn3">[3]</a> format provided so you may see how to substitute into any tagged value, e.g. &lt;​      <sup>inputfile​</sup>​    &gt; in the example given

<a href="#_ftnref4" name="_ftn4">[4]</a> Huge Stock Market Dataset, kaggle.com, accessed November 4, 2019​