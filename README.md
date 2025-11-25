<!DOCTYPE html>
<html>
<head>
<title>Zebra Label Disigner</title>
<link rel="stylesheet"
	href="http://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap.min.css" />
<link rel="stylesheet" href="Styles.css" />
<script
	src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.3/jquery.min.js"></script>
<script
	src="http://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/js/bootstrap.min.js"></script>
<script type="text/javascript" src="LabelElements.js"></script>
<script type="text/javascript" src="Scripts.js"></script>
<script type="text/javascript">
/* Conversion vars (inches, mm, dots and pt) */
var mmpi = 25.4;
var dpi = 203;
var ptpmm = 2.834645669;
var dpmm = dpi / mmpi;

function mm2dots(mm) {
	return Math.round(mm * dpmm);
}

function pt2dots(pt) {
	return mm2dots(pt / ptpmm);
}

/* Label elements */
var defaultElement = {
	title : "New element",
	target : "labelElement_",
	text : "New element",
	"font-size" : {
		measure : 11,
		unit : "pt"
	},
	left : {
		measure : 5,
		unit : "mm"
	},
	top : {
		measure : 5,
		unit : "mm"
	}
};
var labelElements = [];
var label = {
	height : {
		measure : 50,
		unit : "mm"
	},
	width : {
		measure : 50,
		unit : "mm"
	},
	left : {
		measure : 0,
		unit : "mm"
	},
	top : {
		measure : 0,
		unit : "mm"
	},
	elements : labelElements
};

function addElement() {
	return addElement(defaultElement["title"]);
}

function addElement(title) {
	var element = JSON.parse(JSON.stringify(defaultElement));
	var length = labelElements.length;
	element["target"] += length;
	element["title"] = title;
	labelElements[length] = element;
	return element;
}

function getElement(target) {
	for (var i = 0; i < labelElements.length; i++) {
		if (labelElements[i].target == target) {
			return labelElements[i];
		}
	}
}

function editElement(target, property, subproperty, value) {
	var element = getElement(target);
	if (subproperty == "") {
		element[property] = value;
	} else {
		element[property][subproperty] = value;
	}
	return element;
}

function updateFields(target) {
	$(".elementMeasure, #elementText").prop("readonly", false);
	var element = getElement(target);
	$(".elementMeasure").each(function() {
		var property = $(this).attr("id");
		$(this).val(element[property]["measure"]);
	});
	$("#elementText").val(element["text"]);
}
function addToSelect(element) {
	var $option = $("<option>");
	var target = element["target"]
	$option.val(target);
	$option.text(element["title"]);
	$("#target").append($option);
	$("#target").val(target);
}
function addToLabel(element) {
	var $div = $("<div>");
	var fontSize = element["font-size"]["measure"]
			+ element["font-size"]["unit"];
	var x = element["left"]["measure"] + element["left"]["unit"];
	var y = element["top"]["measure"] + element["top"]["unit"];
	$div.attr("class", "labelElement");
	$div.text(element["text"]);
	$div.attr("id", element["target"]);
	$div.css({
		"font-size" : fontSize,
		"left" : x,
		"top" : y
	});
	$("#labelHome").append($div);
}
function updateZPL() {
	var newLine = "\n";
	var coma = ",";
	var zpl = "^XA" + newLine;
	zpl += "^LH" + mm2dots(label["left"]["measure"]) + coma
			+ mm2dots(label["top"]["measure"]) + newLine;
	zpl += "^LL" + mm2dots(label["height"]["measure"]) + newLine;
	zpl += "^PW" + mm2dots(label["width"]["measure"]) + newLine;
	zpl += "^MUd" + newLine;
	for (var i = 0; i < label["elements"].length; i++) {
		var element = label["elements"][i];
		zpl += "^FO" + mm2dots(element["left"]["measure"]) + coma
				+ mm2dots(element["top"]["measure"]);
		zpl += "^A0N" + coma + pt2dots(element["font-size"]["measure"]) + coma;
		zpl += "^CI28";
		zpl += "^FD" + element["text"] + "^FS" + newLine;
	}
	zpl += "^XZ" + newLine;
	$("#zplArea").val(zpl);
}
function init() {
	$("input[type=number]").attr("step", "1");
	$(".labelMeasure").each(function() {
		var property = $(this).attr("property");
		var value = label[property];
		$(this).val(value["measure"]);
		$("#label").css(property, value["measure"] + value["unit"]);
	});
	$(".labelHomeMeasure").each(function() {
		var property = $(this).attr("property");
		var value = label[property];
		$(this).val(value["measure"]);
		$("#labelHome").css(property, value["measure"] + value["unit"]);
	});
	for (var i = 0; i < label["elements"].length; i++) {
		var element = label["elements"][i];
		addToSelect(element);
		addToLabel(element);
	}
	if (label["elements"].length > 0) {
		updateFields($("#target").val());
	} else {
		$(".elementMeasure, #elementText").prop("readonly", true).val("");
	}
	updateZPL();
}
$(document).ready(function() {
	init();
	$(".labelMeasure").change(function() {
		var property = $(this).attr("property");
		var value = $(this).val();
		var unit = label[property]["unit"];
		$("#label").css(property, value + unit);
		label[property]["measure"] = value;
		updateZPL();
	});
	$(".labelHomeMeasure").change(function() {
		var property = $(this).attr("property");
		var value = $(this).val();
		var unit = label[property]["unit"];
		$("#labelHome").css(property, value + unit);
		label[property]["measure"] = value;
		updateZPL();
	});
	$("#target").change(function() {
		var target = $(this).val();
		updateFields(target);
	});
	$("#elementText").change(function() {
		var target = $("#target").val();
		var value = $(this).val();
		$("#" + target).text(value);
		editElement(target, "text", "", value);
		updateZPL();
	});
	$(".elementMeasure").change(function() {
		var target = $("#target").val();
		var property = $(this).attr("id");
		var value = $(this).val();
		var element = editElement(target, property, "measure", value);
		$("#" + target).css(property, value + element[property]["unit"]);
		updateZPL();
	});
	$("#saveElement").click(function() {
		var title = $("#elementTitle").val();
		var element = addElement(title);
		var target = element["target"];
		addToSelect(element);
		addToLabel(element);
		updateFields(target);
		updateZPL();
		$("#elementTitle").val("");
		$("#elementText").focus();
	});
});

</script>
<style>
button {
	float: right;
}

label, input, select {
	float: left;
}

.labelElement {
	position: absolute;
}

#label {
	border: 2px solid;
	margin: 5em auto;
	position: relative;
	overflow: visible;
}

#labelHome {
	position: relative;
	overflow: visible;
}
</style>
</head>
<body>
	<div class="container">
		<form action="#">
			<div class="row">
				<div class="col-md-6">
					<fieldset>
						<legend>Label Settings</legend>
						<div class="row">
							<div class="col-md-6">
								<label for="labelHeight" class="control-label">Height:</label>
							</div>
							<div class="col-md-3">
								<div class="input-group">
									<input type="number" class="form-control labelMeasure"
										property="height" id="labelHeight" /><span
										class="input-group-addon">mm</span>
								</div>
							</div>
						</div>
						<div class="row">
							<div class="col-md-6">
								<label for="labelWidth" class="control-label">Width:</label>
							</div>
							<div class="col-md-3">
								<div class="input-group">
									<input type="number" class="form-control labelMeasure"
										property="width" id="labelWidth" /><span
										class="input-group-addon">mm</span>
								</div>
							</div>
						</div>
						<div class="row">
							<div class="col-md-6">
								<label for="leftMargin" class="control-label">Left
									margin:</label>
							</div>
							<div class="col-md-3">
								<div class="input-group">
									<input type="number" class="form-control labelMeasure"
										property="left" id="leftMargin" /><span
										class="input-group-addon">mm</span>
								</div>
							</div>
						</div>
						<div class="row">
							<div class="col-md-6">
								<label for="topMargin" class="control-label">Top margin:</label>
							</div>
							<div class="col-md-3">
								<div class="input-group">
									<input type="number" class="form-control labelMeasure"
										property="top" id="topMargin" /><span
										class="input-group-addon">mm</span>
								</div>
							</div>
						</div>
					</fieldset>
					<fieldset>
						<legend>Element Settings</legend>
						<div class="row">
							<button type="button" class="btn btn-primary" id="newElement"
								data-toggle="modal" data-target="#newElementModal">New</button>
						</div>
						<div class="row">
							<div class="col-md-6">
								<label for="target" class="control-label">Element:</label>
							</div>
							<div class="col-md-6">
								<select class="form-control" id="target">
								</select>
							</div>
						</div>
						<div class="row">
							<div class="col-md-6">
								<label for="elementText" class="control-label">Text:</label>
							</div>
							<div class="col-md-6">
								<input type="text" class="form-control" id="elementText" />
							</div>
						</div>
						<div class="row">
							<div class="col-md-6">
								<label for="font-size" class="control-label">Font size:</label>
							</div>
							<div class="col-md-3">
								<div class="input-group">
									<input type="number" class="form-control elementMeasure"
										id="font-size" /><span class="input-group-addon">pt</span>
								</div>
							</div>
						</div>
						<div class="row">
							<div class="col-md-6">
								<label for="left" class="control-label">X:</label>
							</div>
							<div class="col-md-3">
								<div class="input-group">
									<input type="number" class="form-control elementMeasure"
										id="left" /><span class="input-group-addon">mm</span>
								</div>
							</div>
						</div>
						<div class="row">
							<div class="col-md-6">
								<label for="top" class="control-label">Y:</label>
							</div>
							<div class="col-md-3">
								<div class="input-group">
									<input type="number" class="form-control elementMeasure"
										id="top" /><span class="input-group-addon">mm</span>
								</div>
							</div>
						</div>
					</fieldset>
				</div>
				<div class="row">
					<div class="col-md-6">
						<fieldset>
							<legend>ZPL</legend>
							<textarea rows="15" cols="50" readonly="readonly" id="zplArea"></textarea>
						</fieldset>
					</div>
				</div>
			</div>
		</form>
		<div class="row">
			<div id="label">
				<div id="labelHome"></div>
			</div>
		</div>
		<footer>
			<p>&copy; The Assemblers' Solutions 2015</p>
		</footer>
	</div>
	<!-- Hidden HTML elements -->
	<div class="modal fade" id="newElementModal" role="dialog"
		aria-labelledby="newElementDialogTitle">
		<div class="modal-dialog">
			<div class="modal-content">
				<div class="modal-header">
					<button type="button" class="close" data-dismiss="modal"
						aria-label="Close">
						<span aria-hidden="false">&times;</span>
					</button>
					<h4 class="modal-title" id="newElementDialogTitle">New Label
						Element</h4>
				</div>
				<div class="modal-body">
					<input type="text" class="form-control" id="elementTitle"
						placeholder="Element title" />
				</div>
				<div class="modal-footer">
					<button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
					<button type="button" class="btn btn-primary" data-dismiss="modal"
						id="saveElement">Save</button>
				</div>
			</div>
		</div>
	</div>
</body>
</html>
