# TextMateSharp
A port of [eclipse/tm4e](https://github.com/eclipse/tm4e) to bring TextMate grammars to dotnet ecosystem.

TextMateSharp uses [Oniguruma](https://github.com/kkos/oniguruma) regex engine bindings.

Instructions about how to build Oniguruma bindings can be found in [`lib/README.md`](https://github.com/danipen/TextMateSharp/tree/master/lib)

## Using
```csharp
        static void Main(string[] args)
        {
            try
            {
                IRegistryOptions options = new LocalRegistryOptions();
                Registry.Registry registry = new Registry.Registry(options);

                List<string> textLines = new List<string>();
                textLines.Add("using static System;");
                textLines.Add("namespace Example");
                textLines.Add("{");
                textLines.Add("}");

                IGrammar grammar = registry.LoadGrammar("source.cs");

                foreach (string line in textLines)
                {
                    Console.WriteLine(string.Format("Tokenizing line: {0}", line));

                    ITokenizeLineResult result = grammar.TokenizeLine(line);

                    foreach (IToken token in result.GetTokens())
                    {
                        int startIndex = (token.StartIndex > line.Length) ?
                            line.Length : token.StartIndex;
                        int endIndex = (token.EndIndex > line.Length) ?
                            line.Length : token.EndIndex;

                        if (startIndex == endIndex)
                            continue;

                        Console.WriteLine(string.Format(
                            "  - token from {0} to {1} -->{2}<-- with scopes {3}",
                            startIndex,
                            endIndex,
                            line.SubstringAtIndexes(startIndex, endIndex),
                            string.Join(",", token.Scopes)));
                    }
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine("ERROR: " + ex.Message);
            }
        }

        class LocalRegistryOptions : IRegistryOptions
        {
            public string GetFilePath(string scopeName)
            {
                string result = Path.GetFullPath(
                    @"../../../../test/testgrammars/csharp.tmLanguage.json");
                return result;
            }

            public ICollection<string> GetInjections(string scopeName)
            {
                return null;
            }

            public StreamReader GetInputStream(string scopeName)
            {
                return new StreamReader(GetFilePath(scopeName));
            }

            public IRawTheme GetTheme()
            {
                return null;
            }
        }
    }
```

OUTPUT:
```
Tokenizing line: using static System;
  -token from 0 to 5 ->using<- with scopes source.cs,keyword.other.using.cs
  -token from 5 to 6 -> <- with scopes source.cs
  -token from 6 to 12 ->static<- with scopes source.cs,keyword.other.static.cs
  -token from 12 to 13 -> <- with scopes source.cs
  -token from 13 to 19 ->System<- with scopes source.cs,storage.type.cs
  -token from 19 to 20 ->;<- with scopes source.cs,punctuation.terminator.statement.cs
Tokenizing line: namespace Example
  -token from 0 to 9 ->namespace<- with scopes source.cs,keyword.other.namespace.cs
  -token from 9 to 10 -> <- with scopes source.cs
  -token from 10 to 17 ->Example<- with scopes source.cs,entity.name.type.namespace.cs
Tokenizing line: {
  -token from 0 to 1 ->{<- with scopes source.cs,punctuation.curlybrace.open.cs
Tokenizing line: }
  -token from 0 to 1 ->}<- with scopes source.cs
```