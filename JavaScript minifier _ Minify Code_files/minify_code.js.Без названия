jQuery(document).ready(function(){
	
	// Menu on mobile
	jQuery('a.menu-icon').on('click', function(e){
		e.preventDefault();
		jQuery('ul.nav').toggle();
	});
	
	/*	Beautify	*/
	if (jQuery('#source').length > 0) {
		
		editor = CodeMirror.fromTextArea(document.getElementById("source"), {
			theme: 'default',
			height: 300,
			lineNumbers: true,
			mode: code_mirror_type
		});
		editor.focus();
		
		jQuery('.CodeMirror').on('click', function(){
			if (jQuery(this).hasClass('selected')) {
				editor.execCommand("singleSelection");
				jQuery(this).removeClass('selected')
			} else {
				editor.execCommand("selectAll");
				jQuery(this).addClass('selected')
			}
			
		});
	}
	
	jQuery('.beautify_submit').on('click', function(e){
		e.preventDefault();
		if ( (editor.getValue() == '') ) {
			alert('Please enter code to editor.');
			return false;
		}
		beautify();
	});
	
	var the = {
		beautify_in_progress: false,
		editor: null // codemirror editor
	};

	function any(a, b) {
		return a || b;
	}

	function unpacker_filter(source) {
		var trailing_comments = '',
			comment = '',
			unpacked = '',
			found = false;

		// cut trailing comments
		do {
			found = false;
			if (/^\s*\/\*/.test(source)) {
				found = true;
				comment = source.substr(0, source.indexOf('*/') + 2);
				source = source.substr(comment.length).replace(/^\s+/, '');
				trailing_comments += comment + "\n";
			} else if (/^\s*\/\//.test(source)) {
				found = true;
				comment = source.match(/^\s*\/\/.*/)[0];
				source = source.substr(comment.length).replace(/^\s+/, '');
				trailing_comments += comment + "\n";
			}
		} while (found);

		var unpackers = [P_A_C_K_E_R, Urlencoded, /*JavascriptObfuscator,*/ MyObfuscate];
		for (var i = 0; i < unpackers.length; i++) {
			if (unpackers[i].detect(source)) {
				unpacked = unpackers[i].unpack(source);
				if (unpacked != source) {
					source = unpacker_filter(unpacked);
				}
			}
		}

		return trailing_comments + source;
	}


	function beautify() {
		if (the.beautify_in_progress) return;

		the.beautify_in_progress = true;
		jQuery('.proccessing').show(0);
		jQuery('.result_message').hide(0);

		var source = editor.getValue(), // jQuery('#source').val(),
			output,
			opts = {};

		opts.indent_size = 1;
		opts.indent_char = opts.indent_size == 1 ? '\t' : ' ';
		opts.max_preserve_newlines = -1;
		opts.preserve_newlines = opts.max_preserve_newlines !== -1;
		opts.keep_array_indentation = false;
		
		opts.break_chained_methods = false;
		
		opts.indent_scripts = 'normal';
		
		opts.brace_style = 'collapse';
		opts.space_before_conditional = true;
		opts.unescape_strings = false;
		opts.wrap_line_length = 0;
		opts.space_after_anon_function = true;

		var function_name = jQuery('#source').attr('rel');
		if (function_name == 'html_beautify') {
			output = html_beautify(source, opts);
		} else if (function_name == 'css_beautify') {
			output = css_beautify(source, opts);
		} else {
			source = unpacker_filter(source);
			output = js_beautify(source, opts);
		}

		editor.setValue(output); //jQuery('#source').val(output);

		jQuery('.proccessing').hide(0);
		jQuery('.result_message').show(0);
		the.beautify_in_progress = false;
	}
	/*	End beautify	*/
	
	//jQuery( "#minify_tabs" ).tabs();
	
	jQuery('.close_message').on('click', function(){
		jQuery(this).parents('.messages').slideToggle('up');
		return false;
	});
	
	// Select all code when focus to code result textarea
	jQuery('.enter_code, .code_result, .from_urls').focus(function(){
		select_all(jQuery(this));
	});
	
	// Select all function
	function select_all(selector) {
		selector.select();
		// Work around Chrome's little problem
    selector.mouseup(function() {
			// Prevent further mouseup intervention
			selector.unbind("mouseup");
			return false;
    });
	}
	
	// Reset the form
	jQuery('.minify_area form input[type="reset"]').on('click', function(){
		
		// Empty editor
		editor.setValue('');
		
		// Hide result message
		jQuery(this).parents('.minify_area').find('.result_message').hide();		
		
		jQuery(this).parents('.minify_area_left').children('form').children('.form-item').children('.form_item_right').children('.add_more_files').each(function(){																																													
			jQuery(this).children('li').each(function(index){
				if (index != 0) {
					// Remove files, urls input
					jQuery(this).remove();
				}
			});			
		});
	});
	
	var form_submit_options = { 
		beforeSubmit: before_submit_minify_form,
		success: minify_submitted,  // post-submit callback 
		dataType:  'json',        // 'xml', 'script', or 'json' (expected server response type) 
		timeout:   120000,
		error: minify_ajax_error
	};
	// The submit form process
	jQuery('.minify_area form').ajaxForm(form_submit_options);
	
	function before_submit_minify_form(formData, jqForm, options){
		var minify_id = '#'+ jQuery('#'+ formData[0].name).parents('.minify_area').attr('id');
		
		// Validate empty
		var enter_code = editor.getValue(); //jQuery(minify_id).find('.enter_code').val();
		
		if ( (enter_code == '') ) {
			alert('Please enter code to editor.');
			return false;
		}		
		
		// Hide result message
		jQuery('.proccessing').show(0);
		jQuery('.result_message').hide(0);
		
		// Show processing bar
		jQuery(minify_id).find('.minify_processing').show(0);
	}
	
	// Error handler
	function minify_ajax_error(error){
		//alert(error.status);
	}
	
	function minify_submitted(data, status, xhr, form) {		

		if (status = 'success') {
			
			// Init jQuery selector
			var minify_id = '#'+ data.type +'_minify';
			var minified_code_textarea = jQuery(minify_id).find('.code_result');
			// Ok
			if (data.status == 'status') {
				
				// Input the minified code to textarea
				//minified_code_textarea.val(data.minified_code);
				editor.setValue(data.minified_code);
				
				// Show minified area
				jQuery(minify_id).find('.copy_code_wrap').show(0);		
				// The result
				jQuery(minify_id).find('.minified_result_detail_wrap').html(data.minified_result);
				// Hide processing bar
				jQuery(minify_id).find('.minify_processing').hide(0);
			}
			
			//result_message.addClass(data.status);
			//result_message.children('span').html(data.message);
			
			// Hide result message
			jQuery('.proccessing').hide(0);
			jQuery('.result_message').show(0);
			// End of JSON process
		}		
	}
	
});