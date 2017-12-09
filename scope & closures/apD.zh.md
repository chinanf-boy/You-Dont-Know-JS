
# 你不知道JS: 范围和闭包

# 附录D: 致谢

我要感谢许多人使这本书的标题和整个系列发生ㄢ

首先,我要感谢我的妻子Christen Simpson和我的两个孩子尼格买提·热合曼和艾米丽,把爸爸一直琢磨计算机ㄢ即使不写书,我痴迷的JavaScript胶水我的眼睛到屏幕远比它应该ㄢ那时候我向我的家人借钱是这些书能深刻而全面地解释JavaScript给读者的原因ㄢI owe my family everything.

我要感谢我的编辑西蒙O'Reilly,即圣劳伦特和Brian MacDonald,以及编辑和营销人员的休息ㄢ他们很好共事,在这个实验中特别乐于接受"开源ℽ的书籍写作ㄡ编辑和制作ㄢ

谢谢你曾经参与制作这本书系列提供编辑的建议和修正,更包括Shelley Powers,Tim Ferro,Evan Borden,Forrest L. Norvell,Jennifer Davis,Jesse Harlin的许多人,和许多其他ㄢ非常感谢Shane Hudson为这个标题写前言ㄢ

谢谢你在社会上无数的人,包括TC39委员会成员,拥有过与我们其他人多的知识,尤其是不能容忍我不停的问题,耐心而详细的探索ㄢJohn David Dalton,"kangax Juriy"Zaytsev,Mathias Bynens,Axel Rauschmayer,Nicholas Zakas,Angus Croll,Reginald Braithwaite,Dave Herman,布兰登·艾奇,Allen Wirfs Brock,Bradley Meck,Domenic Denicola,David Walsh,Tim Disney,Peter van der Zee,Andrea Giammarchi,剑桥,Eric Elliott,和许多人一样,我甚至不能划伤表面ㄢ

这个_你不知道JS_本系列诞生在Kickstarter上,所以我也要感谢我所有的(几乎)500慷慨的支持者,没有他们,这系列的书是不可能发生的: 

> Jan Szpila,Murali Krishnamoorthy,Ryan Joy,Craig Patchett nokiko,,pdqtrader,Dale Fukami,哈特菲尔德,佩雷斯r0drigo[MX],Dan Petitt,Jack Franklin,Andrew Berry,Brian Grinstead,Rob Sutherland,Sergi Meseguer,Phillip Gourley,Mark Watson,Jeff Carouth,Alfredo Sumaran,Martin Sachse,Marcio Barrios,丹,AimelyneM,Matt Sullivan,Delnatte Pierre Antoine,Jake Smith,Eugen Tudorancea,David Trinh,simonstl虹膜,,Ray Daly,Uros Gruber,Justin Myers,Shai Zonis,爸爸妈妈,Devin Clark,Dennis Palmer,Brian Panahi Johnson,Josh Marshall,Marshall,Dennis Kerr,Matt Steele,Erik Slagter,Sacah,Justin Rainbow,Christian Nilsson,Delapouite,D. Pereira,Nicolas Hoizey,George V. Reilly,Dan Reeves,Bruno Laturner,Chad Jennings,Shane King,Jeremiah Lee Cohick,Stan Yamane,Marko Vucinic,od3n,Jim B,斯蒂芬·科林斯,ÆGIRÞorsteinsson,Eric Pederson,Owain,拿芬史密夫,jeanetteurphy,亚历山大伊利斯É,Chris Peterson,Rik Watson,Luke Matthews,Justin Lowery,M·内鲁臣,Vernon Kesner,Chetan Shenoy,Paul Tregoing,Marc Grabanski,Dion Almaer,Andrew Sullivan,Keith Elsass,汤姆·伯克,Brian Ashenfelter,David Stuart,Karl Swedberg,格雷姆,Brandon Hays,John Christopher,Gior,manoj reddy,Chad Smith,Jared Harbour,Minoru TODA,Chris Wigley,Daniel Mee,迈克,handyface,Alex Jahraus,Carl Furrow,Rob Foulkrod,Max Shishkin,Leigh Penny Jr.,Robert Ferguson,Mike van Hoenselaar,Hasse Schougaard,Jeff Adams,Trae Robbins,venkataguru拉,Rolf Langenhuijzen,Jorge Antunes,Alex Koloskov,Hugh Greenish,Tim Jones,Jose Ochoa,Michael Brennan White,Naga Harish Muvva,皮óCZI DáVID,Kitt Hodsden,Paul McGraw,Sascha Goldhofer,Andrew Metcalf,Markus Krogh,Michael Mathews,Matt Jared,Juanfran,Georgie Kirschner,Kenny Lee,Ted Zhang,Amit Pahwa,Dan Raine,Schabse Laks,英巴尔西奈,Michael Tervoort,Alexandre Abreu,Alan Joseph Williams,Cindy Wong,Reg Braithwaite,nicolasd,LocalPCGuy,Jon Friskics,Chris Merriman,John Pena,Jacob Katz,Sue Lockwood,M·祖汉臣,Jeremy Crapsey,10足łowski,nico nuzzaci,Christine Wilks,Hans Bergren,查尔斯,蒙哥马利,Arielבר-לבבFogel,伊万·科列夫,Daniel Campos,Hugh Wood,Christian Bradford,FRéDéRIC哈珀,光网络单元ţ丹波帕,Jeff Trimble,Rupert Wood,Trey Carrico,Pancho Lopez,乔ëL kuijten,Tom A Marra,Jeff Jewiss,Jacob Rios,Paolo Di Stefano,唯一的dad Penades,Chris Gerber,Andrey Dolganov,Wil Moore III,Thomas Martineau,卡里姆,Ben Thouret,Udi Nir,jory carson burson,Nathan L Smith,摩根laupies,Eric Damon Walters,Derry Lozano Hoyland,Geoffrey Wiseman,mkeehner,katiek,Scott MacFarlane,Brian LaShomb,Adrien Mas,christopher ross,Ian Littman,Dan Atkinson,Elliot Jobe,Nick Dozier,Peter Wooley,John Hoover,丹,Martin A. Jackson,andy ennamorato,Fernando HurtadoéH ctor,Paul Seltmann,Melissa Gore,Dave Pollard,杰克·史密斯,Philip Da Silva,盖伊以色列,@巨石,Damian Crawford,Felix Gliesche,四月Carter Grant,海蒂,jim tierney,Andrea Giammarchi,Nico Vignola,Don Jones,Chris Hartjes,Al

这本书系列正在以开放源码的方式制作,包括编辑和制作ㄢ我们欠GitHub报恩做那种事可能为社区!

再次感谢所有我没有名字的人,但是我又感激谁呢?ㄢ愿这本书系列由我们所有人"拥有ℽ,并有助于提高对JavaScript语言的认识和理解,造福于当前和未来的社区贡献者ㄢ
