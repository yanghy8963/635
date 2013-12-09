<?php
function recipeForm_menu() {
    $items = array();
    $items['submitrecipe'] = array(
      'page callback'=>'recipeForm_f1',
      'access arguments' => array('recipe permission'),
      'title'=>'Submit your recipe'
    );

    return $items;
}

function recipeForm_permission(){
    return array(
        'recipe permission' => array(
            'title' => t("recipe permission")
        )
    );
}

function recipeForm_f1() {
    return drupal_get_form('recipeForm_form1');
}

function recipeForm_form1($form_state) {
    $form = array();
    $form['title'] = array(
      '#type'=>'textfield',
      '#title'=> t('Title'),
      '#required' => TRUE
    );
    $form['description'] = array(
      '#type'=>'textarea',
      '#title'=> t('Description'),
      '#required' => TRUE
    );
    $form['ingredient'] = array(
      '#type'=>'textarea',
      '#title'=> t('Ingredient'),
      '#required' => TRUE
    );
    $form['procedure'] = array(
      '#type'=>'textarea',
      '#title'=> t('Procedure'),
      '#required' => TRUE
    );
    $form['recipe_photo'] = array(
        '#type' => 'managed_file',
        '#name' => 'recipe_photo',
        '#title' => t('Your recipe photo'),
        '#size' => 200,
        '#description' => t("Image should be less than 2000 pixels wide."),
        '#upload_location' => 'public://'
    ); 
    
    $form['submit'] = array('#type' => 'submit', '#value' => t('Submit'));

    return $form;
}

function recipeForm_form1_validate($form, $form_state) {
    
}

function recipeForm_form1_submit($form,$form_state) {
    global $user;
    
    $node = new stdclass();
    $node->type = "recipe";
    $node->status = 1;
    $node->promote = 1;
    $node->comment = 2;
    $node->language = LANGUAGE_NONE;
    $node->uid = $user->uid;
    $node->title = $form_state['values']['title'];
    $node->field_description['und'][0]['value'] = $form_state['values']['description'];
    $node->field_ingredient['und'][0]['value'] = $form_state['values']['ingredient'];
    $node->field_procedure['und'][0]['value'] = $form_state['values']['procedure'];
    node_save($node);
    
    if (!empty($form_state['values']['recipe_photo'])) {
        node_load($node->nid);
        $file = file_load($form_state['values']['recipe_photo']);
        $file->status = FILE_STATUS_PERMANENT;
        file_save($file);
        file_usage_add($file, 'recipeForm', 'recipePhoto', $node->nid);
        
        $node->field_photo['und'][] = array(
            'fid' => $file->fid,
            'display' => 1,
            'description' => $node->title . ' Recipe Image',
        );
        node_save($node);
    }
    drupal_set_message("Recipe saved successfully!");
}
