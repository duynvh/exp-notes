# Good articles
1. Back to basics: https://www.joelonsoftware.com/2001/12/11/back-to-basics/
	- Summary: 
		- give some points to handle `string` in `bytes` view
		- show how `malloc` works in C, and why you should always double the size of memory that was previously allocated.
		- you can’t implement the SQL statement **SELECT author FROM books** fast when your data is stored in XML -> how SQL stores data in tables.
	- Homework:
		- For homework, think about why Transmeta chips will always feel sluggish. Or why the original HTML spec for TABLES was so badly designed that large tables on web pages can’t be shown quickly to people with modems. Or about why COM is so dang fast but not when you’re crossing process boundaries. Or about why the NT guys put the display driver into kernelspace instead of userspace.