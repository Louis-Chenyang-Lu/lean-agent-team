# Prompt templates

Replace each `<...>`.

## Code change
```
/team In <file or module>, <change to make>. Done means: <tests or command> pass and <behaviour>. Don't touch <files>.
```

## Analysis
```
/team Analyze <data file> to answer <question>. Write the result to <path> with the method, the key numbers and their caveats.
```

## Writing
```
/team Draft <document type> for <audience> about <topic>, about <length>, saved to <path>. It must cover <points>.
```

## Mechanical batch work (Haiku, no review)
```
/team --haiku <convert / rename / extract> every <file pattern> in <folder> to <target>. Write a short list of what changed to <path>.
```

## Mechanical work you want checked
```
/team --haiku --review Extract <fields> from <files> into <output file>. Every input file must have one row.
```

## 中文示例
```
/team 把 <文件夹> 里的 <内容> 整理成 <格式>，保存到 <路径>。完成标准：<检查方式>。
```
