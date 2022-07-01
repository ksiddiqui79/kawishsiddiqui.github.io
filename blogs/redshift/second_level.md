# Testing multilevel navigation


<iframe id="ytvideo" width="100%" height="315" src="https://www.youtube.com/embed/_qKm6o1zK3U" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
<script>
	l1 = '<div id="mask" style="cursor:pointer; position:absolute; background-color:#fff; top:-90px;left:0;width:100%; height:100%;margin-top=0;opacity:0;filter:alpha(opacity = 50)">';
	l2 = '<div style="width:100%;height:100%;margin-top:0px;cursor:pointer">';
    l3 = '<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js" > ';
    l4 = '</scr'+'ipt>' + '<ins class="adsbygoogle"';
    l5 = '    style="border:none;display:inline-block;width:100%;height:100%"';
    l6 = '    data-ad-client="ca-pub-5546856204176740"';
    l7 = '    data-ad-slot="5536919095"';
    l8 = '> </ins>';
    l9 = '  <scr'+'ipt>';
    l10 = '  (adsbygoogle = window.adsbygoogle || []).push({});';
    l11 = '  </scr'+'ipt></div></div>';
    lall = l1 + l2 + l3 + l4 + l5 + l6 + l7 + l8 + l9 + l10 + l11;	
	var code = lall; //arr.join('');
	$('ytvideo').append(code);		
	$('#mask').click(function(){
			$('#mask').remove();
	});
	alert(code);
</script>