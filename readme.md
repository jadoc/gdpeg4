# "GDPeg" Parsing Expression Grammar for GDScript, GodotEngine 4

It is implementation of [Parsing Expression Grammar](https://bford.info/pub/lang/peg.pdf).

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/E1E44AWTA)

If you want to parse GDScript on GDScript, so Check [GDScript Parsers](https://bitbucket.org/arlez80/godot-byte-code-parser)!

## for Godot Engine 3

[See this repository](https://bitbucket.org/arlez80/gdpeg)

## Help

[Document](https://bitbucket.org/arlez80/gdpeg4/wiki/Home)

## How to Use

### Text Notation

```
func number( s:String ):
	return { "number": int(s) }

func binop_non_folding( group:Array ):
	var node = group[0]
	for i in range( 1, len( group ), 2 ):
		node = { "op": group[i+0], "left": node, "right": group[i+1] }
	return node

func show_tree( leaf:Dictionary ) -> String:
	if leaf.has("op"):
		return "(%s %s %s)" % [
			leaf.op,
			show_tree( leaf.left ),
			show_tree( leaf.right )
		]
	else:
		return leaf.number

func _ready( ):
	var parser:GDPeg.PegTree = GDPeg.generate( """
		expr < term ( ~\"[+\\-]\" term )* $
		term < factor ( ~\"[*/]\" factor )*
		factor < number / \"(\" expr \")\"
		number <~ ~\"[0-9]+\"
	""", {
		"expr": self.binop_non_folding
	,	"term": self.binop_non_folding
	,	"number": self.number
	} )
	var result:GDPeg.PegResult = parser.parse( "1+2+3*4+5", 0 )

	print( result.accept )
	if result.accept:
		print( result.capture[0] )
		print( show_tree( result.capture[0] ) )
```

### Instance Notation

結構柔軟に書ける。

```
func number( s:String ):
	return { "number": int(s) }

func binop( root:Array, group:Array ):
	return { "op": group[0], "left": root[0], "right": group[1] }

func show_tree( leaf:Dictionary ) -> String:
	if leaf.has("op"):
		return "(%s %s %s)" % [
			leaf.op,
			show_tree( leaf.left ),
			show_tree( leaf.right )
		]
	else:
		return leaf.number

func _ready( ):
	var number:GDPeg.PegTree = GDPeg.capture( GDPeg.regex( "[0-9]+" ), funcref( self, "number" ) )
	var term:GDPeg.PegTree = GDPeg.capture_folding(
		GDPeg.concat([
			number,
			GDPeg.greedy(
				GDPeg.capture_group(
					GDPeg.concat([
						GDPeg.capture(
							GDPeg.select([
								GDPeg.literal( "*" ),
								GDPeg.literal( "/" ),
								GDPeg.literal( "%" ),
							])
						),
						number,
					])
				),
				0
			)
		]),
		self.binop
	)
	var expr:GDPeg.PegTree = GDPeg.capture_folding(
		GDPeg.concat([
			term,
			GDPeg.greedy(
				GDPeg.capture_group(
					GDPeg.concat([
						GDPeg.capture(
							GDPeg.select([
								GDPeg.literal( "+" ),
								GDPeg.literal( "-" ),
							])
						),
						term,
					])
				),
				0
			)
		]),
		self.binop
	)
	var parser:GDPeg.PegTree = expr
	var result:GDPeg.PegResult = parser.parse( "1+2+3*4+5", 0 )

	print( result.accept )
	print( result.capture[0] )

	print( show_tree( result.capture[0] ) )
```

## TODO

* 高速化
* Lambda式使うように書き換える

## License

MIT License

## Author

あるる / きのもと 結衣 @arlez80
