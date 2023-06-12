## What is Protocol Buffers and WHY ?

Before diving into deep let me try to summarise what we will be learning going through the post.
- What is serialization and deserialization ? 
- What matters and how to choose between different serialization formats available.
- Why do we even want to use Protocal Buffers, its importance by comparing to two other popular formats which are JSON and XML. 
- Pros and Cons of JSON, XML and PROTOCOL BUFFERS.
- Various tradeoffs between different serialization format.

### What is serialization and deserialization
![alt text](/docs/assets/1.png)
- So here on screen we have an object and this object could be anything , for example  a user account with an email and a password. 
- So we could save this into a file, somewhere in computer memory or in a database somewhere in the cloud. We are just going to extract the data that is inside this object and write that data in one or multiple storage mediums

![alt text](/docs/assets/2.png)
- So the bottomline is that serialization is about saving data so that we can reuse it later. There is more to it, this is just the baseline.

![alt text](/docs/assets/3.png)
- So the idea here is just the reverse, we are going to read our storage medium and then we are going to field the data back to our object. This is called deserialization.

### There are actually three major issues that serialization and deserialization are solving. 

![alt text](/docs/assets/5.png)
- `Language Agnosticism` 
    - process of transforming our object data so that the data is easily accessible from different programming languages.
    - We want our data to be accessible in python, accessible in JS, in c++, and any other languages.
    - For example we have a machine learning application in python, webiste in java and a game in c++, they need to share sata with each other.

![alt text](/docs/assets/6.png)
- `Communication`
    - We need multiple machines to be able to communicate and share data.
    - So on top of language agnosticism , we want our data to not be dependent on any operating system.
    - We want our data to be as small as possible so that we can send fata fast over the network and we can receive it fast over the network. Saving network bandwidth and storage dpce.

![alt text](/docs/assets/7.png)
- `Object Relationship`
    - One object may have relationship with other object. 
    - It is important that we serialize these relationships so that any other program that is reusing our data will work the same way as expected.
    - It helps in maintaining consistency across multiple machines and multiple programs.

![alt text](/docs/assets/8.png)

























Due to a plugin called `jekyll-titles-from-headings` which is supported by GitHub Pages by default. The above header (in the markdown file) will be automatically used as the pages title.

If the file does not start with a header, then the post title will be derived from the filename.

This is a sample blog post. You can talk about all sorts of fun things here.

---

### This is a header

#### Some T-SQL Code

```tsql
SELECT This, [Is], A, Code, Block -- Using SSMS style syntax highlighting
    , REVERSE('abc')
FROM dbo.SomeTable s
    CROSS JOIN dbo.OtherTable o;
```

#### Some PowerShell Code

```powershell
Write-Host "This is a powershell Code block";

# There are many other languages you can use, but the style has to be loaded first

ForEach ($thing in $things) {
    Write-Output "It highlights it using the GitHub style"
}
```
