# CPT-CustomHTMLRawForm
Custom Post Type and adding custom form field raw HTML with basic repeater for WordPress

<h4>Custom Post Type | Standard Fields</h4>

```PHP
// Register Custom Post Type
function my_custom_post_type() {
    $args = array(
        'public'    => true,
        'label'     => 'Books',
        'supports'  => array('title', 'editor', 'thumbnail'),
        'menu_icon' => 'dashicons-book',
    );
    register_post_type('book', $args);
}
add_action('init', 'my_custom_post_type');

// Add Custom Meta Box
function my_add_custom_meta_box() {
    add_meta_box(
        'my_custom_meta_box',           // ID of the meta box
        'Book Details',                 // Title of the meta box
        'my_custom_meta_box_callback',  // Callback function
        'book',                         // Custom post type
        'normal',                       // Context (normal, side, advanced)
        'high'                          // Priority (high, low)
    );
}
add_action('add_meta_boxes', 'my_add_custom_meta_box');

// Meta Box Callback Function
function my_custom_meta_box_callback($post) {
    // Nonce field for security
    wp_nonce_field('my_custom_meta_box_nonce', 'my_custom_meta_box_nonce_field');

    // Retrieve existing values from the database
    $author = get_post_meta($post->ID, '_my_custom_author', true);
    $year = get_post_meta($post->ID, '_my_custom_year', true);

    // Display the fields
    ?>
    <p>
        <label for="my_custom_author">Author:</label>
        <input type="text" id="my_custom_author" name="my_custom_author" value="<?php echo esc_attr($author); ?>" size="25" />
    </p>
    <p>
        <label for="my_custom_year">Publication Year:</label>
        <input type="text" id="my_custom_year" name="my_custom_year" value="<?php echo esc_attr($year); ?>" size="25" />
    </p>
    <?php
}

// Save Meta Box Data
function my_save_custom_meta_box_data($post_id) {
    // Check if nonce is set
    if (!isset($_POST['my_custom_meta_box_nonce_field'])) {
        return;
    }

    // Verify the nonce
    if (!wp_verify_nonce($_POST['my_custom_meta_box_nonce_field'], 'my_custom_meta_box_nonce')) {
        return;
    }

    // Check if this is an autosave
    if (defined('DOING_AUTOSAVE') && DOING_AUTOSAVE) {
        return;
    }

    // Check user permissions
    if (!current_user_can('edit_post', $post_id)) {
        return;
    }

    // Sanitize and save data
    if (isset($_POST['my_custom_author'])) {
        $author = sanitize_text_field($_POST['my_custom_author']);
        update_post_meta($post_id, '_my_custom_author', $author);
    }

    if (isset($_POST['my_custom_year'])) {
        $year = sanitize_text_field($_POST['my_custom_year']);
        update_post_meta($post_id, '_my_custom_year', $year);
    }
}
add_action('save_post', 'my_save_custom_meta_box_data');

```

<h4>Custom Post Type | With Basic Repeater</h4>

```PHP
// Register Custom Post Type
function my_custom_post_type() {
    $args = array(
        'public'    => true,
        'label'     => 'Books',
        'supports'  => array('title', 'editor', 'thumbnail'),
        'menu_icon' => 'dashicons-book',
    );
    register_post_type('book', $args);
}
add_action('init', 'my_custom_post_type');

// Add Meta Box with Repeater Fields
function my_add_custom_meta_box() {
    add_meta_box(
        'my_custom_meta_box',
        'Book Details',
        'my_custom_meta_box_callback',
        'book',
        'normal',
        'high'
    );
}
add_action('add_meta_boxes', 'my_add_custom_meta_box');

// Meta Box Callback Function
function my_custom_meta_box_callback($post) {
    // Nonce field for security
    wp_nonce_field('my_custom_meta_box_nonce', 'my_custom_meta_box_nonce_field');

    // Retrieve existing values from the database
    $chapters = get_post_meta($post->ID, '_my_custom_chapters', true);

    // Display the fields
    ?>
    <div id="my-custom-repeater">
        <div id="repeater-fields">
            <?php
            if (!empty($chapters)) {
                foreach ($chapters as $index => $chapter) {
                    ?>
                    <div class="repeater-item">
                        <p>
                            <label for="my_custom_chapters_<?php echo $index; ?>_title">Chapter Title:</label>
                            <input type="text" id="my_custom_chapters_<?php echo $index; ?>_title" name="my_custom_chapters[<?php echo $index; ?>][title]" value="<?php echo esc_attr($chapter['title']); ?>" size="25" />
                        </p>
                        <p>
                            <label for="my_custom_chapters_<?php echo $index; ?>_page">Page Number:</label>
                            <input type="text" id="my_custom_chapters_<?php echo $index; ?>_page" name="my_custom_chapters[<?php echo $index; ?>][page]" value="<?php echo esc_attr($chapter['page']); ?>" size="25" />
                        </p>
                        <button type="button" class="remove-chapter">Remove</button>
                    </div>
                    <?php
                }
            } else {
                // Default empty repeater fields
                ?>
                <div class="repeater-item">
                    <p>
                        <label for="my_custom_chapters_0_title">Chapter Title:</label>
                        <input type="text" id="my_custom_chapters_0_title" name="my_custom_chapters[0][title]" value="" size="25" />
                    </p>
                    <p>
                        <label for="my_custom_chapters_0_page">Page Number:</label>
                        <input type="text" id="my_custom_chapters_0_page" name="my_custom_chapters[0][page]" value="" size="25" />
                    </p>
                    <button type="button" class="remove-chapter">Remove</button>
                </div>
                <?php
            }
            ?>
        </div>
        <button type="button" id="add-chapter">Add Chapter</button>
    </div>

    <script type="text/javascript">
    jQuery(document).ready(function($) {
        let index = <?php echo !empty($chapters) ? count($chapters) : 1; ?>;
        
        $('#add-chapter').on('click', function() {
            $('#repeater-fields').append(`
                <div class="repeater-item">
                    <p>
                        <label for="my_custom_chapters_${index}_title">Chapter Title:</label>
                        <input type="text" id="my_custom_chapters_${index}_title" name="my_custom_chapters[${index}][title]" value="" size="25" />
                    </p>
                    <p>
                        <label for="my_custom_chapters_${index}_page">Page Number:</label>
                        <input type="text" id="my_custom_chapters_${index}_page" name="my_custom_chapters[${index}][page]" value="" size="25" />
                    </p>
                    <button type="button" class="remove-chapter">Remove</button>
                </div>
            `);
            index++;
        });

        $(document).on('click', '.remove-chapter', function() {
            $(this).closest('.repeater-item').remove();
        });
    });
    </script>
    <?php
}

// Save Meta Box Data
function my_save_custom_meta_box_data($post_id) {
    // Check if nonce is set
    if (!isset($_POST['my_custom_meta_box_nonce_field'])) {
        return;
    }

    // Verify the nonce
    if (!wp_verify_nonce($_POST['my_custom_meta_box_nonce_field'], 'my_custom_meta_box_nonce')) {
        return;
    }

    // Check if this is an autosave
    if (defined('DOING_AUTOSAVE') && DOING_AUTOSAVE) {
        return;
    }

    // Check user permissions
    if (!current_user_can('edit_post', $post_id)) {
        return;
    }

    // Sanitize and save repeater data
    if (isset($_POST['my_custom_chapters']) && is_array($_POST['my_custom_chapters'])) {
        $chapters = array();
        foreach ($_POST['my_custom_chapters'] as $chapter) {
            $chapters[] = array(
                'title' => sanitize_text_field($chapter['title']),
                'page'  => sanitize_text_field($chapter['page']),
            );
        }
        update_post_meta($post_id, '_my_custom_chapters', $chapters);
    } else {
        delete_post_meta($post_id, '_my_custom_chapters');
    }
}
add_action('save_post', 'my_save_custom_meta_box_data');


```
