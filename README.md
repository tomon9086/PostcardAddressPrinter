# 宛名印刷プログラム
## 動機
* 今年度から大学入学に伴って一人暮らしをするようになり、今まで親に丸投げしていた年賀状の宛名印刷を自分でやらざるを得なくなった。
* 「筆まめ」「筆ぐるめ」などの既製品も市販されているが、常用のパソコンは低スペックであるし、そうでなくとも年1回、ほぼ年賀状のときのためだけに専用ソフトを買う金とパソコンのディスク容量が惜しかった。

## 目標
* 氏名、住所等が記録されたCSVファイル(実家の筆ぐるめから出力が可能)から、宛名を印刷する。
* 住所録から消すことなく、出さない人を簡単にスキップできる。
* 敬称を変えられる。
* 連名にできる。
* 自動で改行を適切な場所にする。例えば、「東京都新宿区神↩ 楽坂1-3」を「東京都新宿区↩ 神楽坂1-3」と自動補正する。(難)
* プレビューが容易。
* 情報漏洩しない。
* 正規表現の練習がしたい。(自己満足)
* 印刷範囲の自動選択

## 制作したものの概要

[<span style="font-size: 1.5em;">宛名印刷プログラム</span>](https://tomon9086.github.io/PostcardAddressPrinter/)

* 開発言語
	* JavaScript
	* (HTML)
	* (CSS)
* ソースコード  
	[GitHub](https://github.com/tomon9086/PostcardAddressPrinter)にもあります。ここにはJavaScriptのみ載せておきます。
	
	```javascript	
	const defaultResolution = 6;
	var resolution = defaultResolution;
	var kind = "";
	var source = "";
	hagakiOnChange();
	if(window.File && window.FileReader && window.FileList && window.Blob){
		document.getElementById("selectFile").addEventListener("change", function(evt){
			var file = evt.target.files && evt.target.files[0];
			// console.log(evt.target.files);
			if(file){
				var charCode = "";
				if(document.getElementById("utf8").checked){
					charCode = "utf-8";
				}else if(document.getElementById("shiftJis").checked){
					charCode = "shift-jis";
				}else{
					alert("Fatal Error!");
					throw new Error("fatal error");
				}
				var reader = new FileReader();
				try{
					reader.readAsText(file, charCode);
				}catch(error){
					alert("File's extension should be text or csv.");
				}
				reader.onload = function(ev){
					source = reader.result;
					// console.log(source);
					hagakiGenerator(kind, source);
				}
			}
		});
	}else{
		alert('The File APIs are not fully supported in this browser.');
	}

	function hagakiOnChange(opt){
		if(document.getElementById("normal").checked){
			kind = "normal";
			Array.prototype.forEach.call(document.getElementsByClassName("sheet"), function(v, i){
				v.setAttribute("style", "width: 100mm; height: 148mm; background-image: url(hagaki.png); background-size: cover; page-break-after: always;");
			});
		}else if(document.getElementById("nenga").checked){
			kind = "nenga";
			Array.prototype.forEach.call(document.getElementsByClassName("sheet"), function(v, i){
				v.setAttribute("style", "width: 100mm; height: 148mm; background-image: url(nenga.png); background-size: cover; page-break-after: always;");
			});
		}
		if(!opt){
			if(source){
				hagakiGenerator(kind, source);
			}else{
			}
		}else{
		}
	}

	function resolutionSet(){
		var inputValue = document.getElementById("resolutionSetting").value;
		if(inputValue >= 30 && inputValue < 10000){ resolution = defaultResolution * inputValue * 0.01; }
		if(source){ hagakiGenerator(kind, source); }
	}

	function resetButtonOnClick(){
		var select = confirm("Do you want to Reset?");
		if(select){
			location.reload();
		}else{
		}
	}

	function hagakiGenerator(kind, source){
		document.getElementById("printAreaSet").innerHTML = '<section class="sheet" id="printArea"></section>';
		var addressArray = [];
		var newLineRegex = /(.*)[\n|\r|\r\n]/g;
		var addressBookRegex = /(.+)[ |　]+(.+)[ |　|\t]*,[ |　|\t]*(\d\d\d)-(\d\d\d\d)[ |　|\t]*,[ |　|\t]*(.+)[ |　|\t]*,[ |　|\t]*([1-9])/m;
		var receiverArray = source.match(newLineRegex);
		receiverArray.forEach(function(v, i){
			if(v.match(addressBookRegex) !== null){
				addressArray.push(v.match(addressBookRegex));
			}
		});
		console.log(addressArray);
		if(addressArray.length > 2001){
			alert("Too many addresses imported. (> 2000)");
			return;
		}

		var hyphenRegex = /[-|−|―|ー|‐|ｰ|－]/g;
		addressArray.forEach(function(v, i){
			v[5] = v[5].replace(hyphenRegex, "ǀ");
		});

		var exportString = "";
		for(let i = 1; i < addressArray.length; i++){
			exportString += '<section class="sheet" id="printArea"><canvas class="sheetCanvas" id="hagaki_' + i + '"></canvas></section>';
		}
		document.getElementById("printArea").outerHTML = exportString;

		hagakiOnChange(1);
		
		var zipcodeFrom = "";
		var addressFrom = "";
		var nameFrom = "";
		addressArray.forEach(function(v, i){
			if(i !== 0 && v !== null){
				document.getElementById("hagaki_" + i).setAttribute("width", 100 * resolution);
				document.getElementById("hagaki_" + i).setAttribute("height", 148 * resolution);
				var canvas = document.getElementById("hagaki_" + i);
				var canvasText = document.getElementById("hagaki_" + i).getContext("2d");
			}
			if(i === 0 && v !== null){
				nameFrom = {last: v[1], first: v[2]};
				zipcodeFrom = v[3] + v[4];
				addressFrom = v[5];
			}else if(v !== null){
				zipcodeToDisplay(v[3] + v[4], canvasText);
				addressToDisplay(v[5], canvasText);
				nameToDisplay(v[1], v[2], "様", canvasText);
				zipcodeFromDisplay(zipcodeFrom, kind, canvasText);
				addressFromDisplay(addressFrom, kind, canvasText);
				nameFromDisplay(nameFrom.last, nameFrom.first, kind, canvasText);
			}else if(i === 0 && v === null){
				nameFrom = "";
				zipcodeFrom = "";
				addressFrom = "";
			}else{
				
			}
		});
	}

	function zipcodeToDisplay(str, canvasText){
		canvasText.font = 9 * resolution + "px Arial";
		canvasText.textAlign = "center";
		canvasText.fillText(str[0], 46.7 * resolution, 18.3 * resolution);
		canvasText.fillText(str[1], 53.7 * resolution, 18.3 * resolution);
		canvasText.fillText(str[2], 60.7 * resolution, 18.3 * resolution);
		canvasText.fillText(str[3], 68.5 * resolution, 18.3 * resolution);
		canvasText.fillText(str[4], 75.2 * resolution, 18.3 * resolution);
		canvasText.fillText(str[5], 82.2 * resolution, 18.3 * resolution);
		canvasText.fillText(str[6], 89.2 * resolution, 18.3 * resolution);
	}
	function zipcodeFromDisplay(str, kind, canvasText){
		if(kind === "nenga"){
			correction = -15.3 * resolution;
		}else{
			correction = 0 * resolution;
		}
		canvasText.font = 6.5 * resolution + "px Arial";
		canvasText.textAlign = "center";
		canvasText.fillText(str[0], 6.5 * resolution, 145.6 * resolution + correction);
		canvasText.fillText(str[1], 10.5 * resolution, 145.6 * resolution + correction);
		canvasText.fillText(str[2], 14.5 * resolution, 145.6 * resolution + correction);
		canvasText.fillText(str[3], 19.6 * resolution, 145.6 * resolution + correction);
		canvasText.fillText(str[4], 23.6 * resolution, 145.6 * resolution + correction);
		canvasText.fillText(str[5], 27.6 * resolution, 145.6 * resolution + correction);
		canvasText.fillText(str[6], 31.6 * resolution, 145.6 * resolution + correction);
	}
	function addressToDisplay(str, canvasText){
		const fontSize = 7 * resolution;
		const whenNewLine = 110 * resolution;
		var count = 0;
		canvasText.font = fontSize + "px Arial";
		canvasText.textAlign = "center";
		Array.prototype.forEach.call(str, function(v, i){
			if(fontSize * (i + 1) < whenNewLine * 1){
				canvasText.fillText(v, 87 * resolution, 33 * resolution + fontSize * i);
				count++;
			}else if(i < count * 2){
				canvasText.fillText(v, 87 * resolution - (fontSize * 1), 33 * resolution + fontSize * (i % count));
			}else if(i < count * 3){
				canvasText.fillText(v, 87 * resolution - (fontSize * 2), 33 * resolution + fontSize * (i % count));
			}else if(i < count * 4){
				canvasText.fillText(v, 87 * resolution - (fontSize * 3), 33 * resolution + fontSize * (i % count));
			}
		});
	}
	function addressFromDisplay(str, kind, canvasText){
		if(kind === "nenga"){
			correction = -15.3 * resolution;
		}else{
			correction = 0 * resolution
		}
		const fontSize = 4.5 * resolution;
		const whenNewLine = 60 * resolution;
		var count = 0;
		canvasText.font = fontSize + "px Arial";
		canvasText.textAlign = "center";
		Array.prototype.forEach.call(str, function(v, i){
			if(fontSize * (i + 1) < whenNewLine * 1){
				canvasText.fillText(v, 30 * resolution, 83.6 * resolution + fontSize * i + correction);
				count++;
			}else if(fontSize * (i + 1) < whenNewLine * 2){
				canvasText.fillText(v, 30 * resolution - fontSize, 83.6 * resolution + fontSize * (i % count) + correction);
			}
		});
	}
	function nameToDisplay(last, first, honorific, canvasText){
		const fontSize = 13 * resolution;
		const letterSpacing = 4 * resolution;
		const space = 6 * resolution;
		canvasText.font = fontSize + "px Arial";
		canvasText.textAlign = "center";
		var wholePix = last.length * (fontSize + letterSpacing) - letterSpacing + space + first.length * (fontSize + letterSpacing) - letterSpacing + space + honorific.length * (fontSize + letterSpacing) - letterSpacing;
		var pixLastStart = 93 * resolution - wholePix / 2;
		var pixFirstStart = pixLastStart + last.length * (fontSize + letterSpacing) - letterSpacing + space;
		var pixHonorificStart = pixFirstStart + first.length * (fontSize + letterSpacing) - letterSpacing + space;
		Array.prototype.forEach.call(last, function(v, i){
			canvasText.fillText(v, 50 * resolution, pixLastStart + (fontSize + letterSpacing) * i - letterSpacing);
		});
		Array.prototype.forEach.call(first, function(v, i){
			canvasText.fillText(v, 50 * resolution, pixFirstStart + (fontSize + letterSpacing) * i - letterSpacing);
		});
		Array.prototype.forEach.call(honorific, function(v, i){
			canvasText.fillText(v, 50 * resolution, pixHonorificStart + (fontSize + letterSpacing) * i - letterSpacing);
		});
	}
	function nameFromDisplay(last, first, kind, canvasText){
		if(kind === "nenga"){
			correction = -15.3 * resolution;
		}else{
			correction = 0 * resolution;
		}
		const fontSize = 6 * resolution;
		const letterSpacing = 1 * resolution;
		const space = 2 * resolution;
		canvasText.font = fontSize + "px Arial";
		canvasText.textAlign = "center";
		var wholePix = last.length * (fontSize + letterSpacing) - letterSpacing + space + first.length * (fontSize + letterSpacing) - letterSpacing;
		var pixLastStart = 123.6 * resolution + correction - wholePix / 2;
		var pixFirstStart = pixLastStart + last.length * (fontSize + letterSpacing) - letterSpacing + space;

		Array.prototype.forEach.call(last, function(v, i){
			canvasText.fillText(v, 13.5 * resolution, pixLastStart + (fontSize + letterSpacing) * i - letterSpacing);
		});
		Array.prototype.forEach.call(first, function(v, i){
			canvasText.fillText(v, 13.5 * resolution, pixFirstStart + (fontSize + letterSpacing) * i - letterSpacing);
		});
	}
	```
* 参考ページ
	* [そろそろ真面目に、HTMLで帳票を描く話をしようか](http://qiita.com/cognitom/items/d39d5f19054c8c8fd592)
	* [CSSだけで縦書きを表現する](http://sterfield.co.jp/designer/css%E3%81%A0%E3%81%91%E3%81%A7%E7%B8%A6%E6%9B%B8%E3%81%8D%E3%82%92%E8%A1%A8%E7%8F%BE%E3%81%99%E3%82%8B/)

### このプログラムでできること
* 一定の記法に沿って記録された住所録(.txtや.csv)を読み込んでビューを自動生成する。
* それらをブラウザの印刷機能から用紙サイズをはがきに指定することで、宛名印刷ができる。

### 特長
* HTMLとJavaScriptなので、Web上で公開すれば、誰もがOSを問わず宛名印刷ができる。
* 住所録がテキスト形式や、CSV形式と軽い。
* 1度に2000枚の印刷に対応している。
	* canvasで生成するアップデート後、少々印刷プレビューの読み込みが遅くなった。一度に印刷できる枚数は環境に依存する。
* 住所録を読み込んでビュー生成まで、すべてクライアントサイドで処理をしているため、情報漏洩の危険が極めて少ない。

### 工夫
* ビュー生成は7000枚を超えて問題なく確認しているが、2000枚を超えるとブラウザの印刷プレビューの表示に10秒以上かかり、実用的でないと判断し、2000枚を超える住所録を選択するとアラートを出して受け付けない仕様にした。
* 読み込んだファイルがテキストとして取り込めなかった場合(画像を選択した場合など)に出るエラーをtry-catch文で受け取ることで、エラーによってプログラムが停止することを防いだ。
* 普通はがきと年賀はがきではお年玉くじの都合上差出人の郵便番号枠が異なるので、これら2種のはがきに対応した。
* 住所録ファイルを作成したOSによって文字コードが異なる場合があるため、UTF-8とShift-JISを選択できるようにした。

* もともとHTMLタグとCSSで文字(住所、宛名等)を配置していたが、ブラウザやOSに大きく影響されることが判明したため、はがきと同じ大きさのcanvasに文字を描画することでそのような依存性を解消した(canvas内の絶対的な座標が利用できるため)。
* canvasで生成すると画像扱いとなり、動作が重くなるので、環境によって調節できるようにcanvasの解像度の調節バーを設けた。
* スペースの半角スペース・全角スペース・タブ対応はもちろん、ハイフンも7種類の文字に対応して、なるべく誤表示が起こらないように努めた。


## 実際に使ってみて
実際に2017年の年賀状の宛名をこのプログラムで生成し、印刷してみた。<br>
macOSで作成した住所録をWindowsに転送してWindows上のChromeから印刷したが、郵便番号が若干上にずれたこと以外何も問題はなかった。

### 今後の改善点
* 住所録の最後の行に改行を入れないと最終行が読み込まれない仕様があって、ユーザーフレンドリーでない。
* もうちょっとページ自体のスタイルを華やかにしたい。
* やっぱりいつかは改行の自動調整を実装したい。