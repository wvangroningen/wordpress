diff --git a/strato-job-manager/strato-job-manager.php b/strato-job-manager/strato-job-manager.php
new file mode 100644
index 0000000000000000000000000000000000000000..5bbc57cbbd338a04345869954170d73c53310edb
--- /dev/null
+++ b/strato-job-manager/strato-job-manager.php
@@ -0,0 +1,880 @@
+<?php
+/**
+ * Plugin Name:       Strato Job Manager
+ * Plugin URI:        https://example.com/strato-job-manager
+ * Description:       Een backend plug-in voor het beheren van vacatures en kandidaten met sollicitatieformulier en aangepaste lay-out.
+ * Version:           1.0.0
+ * Author:            OpenAI Assistant
+ * License:           GPL-2.0-or-later
+ * License URI:       https://www.gnu.org/licenses/gpl-2.0.html
+ * Text Domain:       strato-job-manager
+ * Domain Path:       /languages
+ */
+
+if ( ! defined( 'ABSPATH' ) ) {
+    exit;
+}
+
+if ( ! class_exists( 'Strato_Job_Manager' ) ) {
+    class Strato_Job_Manager {
+        /**
+         * Singleton instance.
+         *
+         * @var Strato_Job_Manager
+         */
+        protected static $instance = null;
+
+        /**
+         * Vacancy meta fields definition.
+         *
+         * @var array<string, array<string, string>>
+         */
+        protected $vacancy_fields = [];
+
+        /**
+         * Candidate meta fields definition.
+         *
+         * @var array<string, array<string, string>>
+         */
+        protected $candidate_fields = [];
+
+        /**
+         * Form errors.
+         *
+         * @var array<int, string>
+         */
+        protected $form_errors = [];
+
+        /**
+         * Form success flag.
+         *
+         * @var bool
+         */
+        protected $form_success = false;
+
+        /**
+         * Initialise plugin.
+         */
+        private function __construct() {
+            $this->vacancy_fields = [
+                'strato_vacancy_reference'  => [
+                    'label' => __( 'Referentienummer', 'strato-job-manager' ),
+                    'type'  => 'text',
+                ],
+                'strato_vacancy_location'   => [
+                    'label' => __( 'Locatie', 'strato-job-manager' ),
+                    'type'  => 'text',
+                ],
+                'strato_vacancy_hourly'     => [
+                    'label' => __( 'Uurloon', 'strato-job-manager' ),
+                    'type'  => 'text',
+                ],
+                'strato_vacancy_monthly'    => [
+                    'label' => __( 'Maandsalaris', 'strato-job-manager' ),
+                    'type'  => 'text',
+                ],
+                'strato_vacancy_contract'   => [
+                    'label' => __( 'Contractduur', 'strato-job-manager' ),
+                    'type'  => 'text',
+                ],
+                'strato_vacancy_hours'      => [
+                    'label' => __( 'Uren per week', 'strato-job-manager' ),
+                    'type'  => 'text',
+                ],
+                'strato_vacancy_start_date' => [
+                    'label' => __( 'Startdatum', 'strato-job-manager' ),
+                    'type'  => 'date',
+                ],
+                'strato_vacancy_end_date'   => [
+                    'label' => __( 'Einddatum', 'strato-job-manager' ),
+                    'type'  => 'date',
+                ],
+                'strato_vacancy_employment' => [
+                    'label' => __( 'Dienstverband', 'strato-job-manager' ),
+                    'type'  => 'text',
+                ],
+                'strato_vacancy_skills'     => [
+                    'label' => __( 'Vereiste vaardigheden', 'strato-job-manager' ),
+                    'type'  => 'textarea',
+                ],
+                'strato_vacancy_benefits'   => [
+                    'label' => __( 'Arbeidsvoorwaarden', 'strato-job-manager' ),
+                    'type'  => 'textarea',
+                ],
+            ];
+
+            $this->candidate_fields = [
+                'strato_candidate_email'   => [
+                    'label' => __( 'E-mailadres', 'strato-job-manager' ),
+                    'type'  => 'email',
+                ],
+                'strato_candidate_phone'   => [
+                    'label' => __( 'Telefoonnummer', 'strato-job-manager' ),
+                    'type'  => 'text',
+                ],
+                'strato_candidate_message' => [
+                    'label' => __( 'Motivatie', 'strato-job-manager' ),
+                    'type'  => 'textarea',
+                ],
+                'strato_candidate_cv'      => [
+                    'label' => __( 'CV', 'strato-job-manager' ),
+                    'type'  => 'file',
+                ],
+                'strato_candidate_vacancy' => [
+                    'label' => __( 'Vacature', 'strato-job-manager' ),
+                    'type'  => 'number',
+                ],
+            ];
+
+            add_action( 'init', [ $this, 'register_post_types' ] );
+            add_action( 'init', [ $this, 'handle_application_form' ] );
+            add_action( 'init', [ $this, 'register_shortcodes' ] );
+            add_action( 'add_meta_boxes', [ $this, 'register_meta_boxes' ] );
+            add_action( 'save_post', [ $this, 'save_post_meta' ] );
+            add_action( 'admin_enqueue_scripts', [ $this, 'admin_assets' ] );
+            add_action( 'wp_enqueue_scripts', [ $this, 'frontend_assets' ] );
+            add_filter( 'the_content', [ $this, 'append_vacancy_content' ] );
+            add_action( 'manage_strato_vacancy_posts_custom_column', [ $this, 'render_vacancy_columns' ], 10, 2 );
+            add_filter( 'manage_strato_vacancy_posts_columns', [ $this, 'register_vacancy_columns' ] );
+            add_action( 'manage_strato_candidate_posts_custom_column', [ $this, 'render_candidate_columns' ], 10, 2 );
+            add_filter( 'manage_strato_candidate_posts_columns', [ $this, 'register_candidate_columns' ] );
+            add_action( 'restrict_manage_posts', [ $this, 'admin_filters' ] );
+            add_action( 'pre_get_posts', [ $this, 'apply_admin_filters' ] );
+            add_action( 'plugins_loaded', [ $this, 'load_textdomain' ] );
+            register_activation_hook( __FILE__, [ 'Strato_Job_Manager', 'activate' ] );
+            register_deactivation_hook( __FILE__, [ 'Strato_Job_Manager', 'deactivate' ] );
+        }
+
+        /**
+         * Get singleton instance.
+         */
+        public static function instance() {
+            if ( null === self::$instance ) {
+                self::$instance = new self();
+            }
+
+            return self::$instance;
+        }
+
+        /**
+         * Plugin activation callback.
+         */
+        public static function activate() {
+            $instance = self::instance();
+            $instance->register_post_types();
+            flush_rewrite_rules();
+        }
+
+        /**
+         * Plugin deactivation callback.
+         */
+        public static function deactivate() {
+            flush_rewrite_rules();
+        }
+
+        /**
+         * Load plugin textdomain.
+         */
+        public function load_textdomain() {
+            load_plugin_textdomain( 'strato-job-manager', false, dirname( plugin_basename( __FILE__ ) ) . '/languages/' );
+        }
+
+        /**
+         * Register custom post types.
+         */
+        public function register_post_types() {
+            register_post_type(
+                'strato_vacancy',
+                [
+                    'labels'       => [
+                        'name'               => __( 'Vacatures', 'strato-job-manager' ),
+                        'singular_name'      => __( 'Vacature', 'strato-job-manager' ),
+                        'add_new'            => __( 'Nieuwe vacature', 'strato-job-manager' ),
+                        'add_new_item'       => __( 'Nieuwe vacature toevoegen', 'strato-job-manager' ),
+                        'edit_item'          => __( 'Vacature bewerken', 'strato-job-manager' ),
+                        'new_item'           => __( 'Nieuwe vacature', 'strato-job-manager' ),
+                        'view_item'          => __( 'Vacature bekijken', 'strato-job-manager' ),
+                        'search_items'       => __( 'Vacatures zoeken', 'strato-job-manager' ),
+                        'not_found'          => __( 'Geen vacatures gevonden', 'strato-job-manager' ),
+                        'not_found_in_trash' => __( 'Geen vacatures gevonden in prullenbak', 'strato-job-manager' ),
+                        'all_items'          => __( 'Alle vacatures', 'strato-job-manager' ),
+                        'menu_name'          => __( 'Vacatures', 'strato-job-manager' ),
+                    ],
+                    'public'       => true,
+                    'has_archive'  => true,
+                    'rewrite'      => [ 'slug' => 'vacatures' ],
+                    'supports'     => [ 'title', 'editor', 'excerpt', 'thumbnail', 'author' ],
+                    'show_in_rest' => true,
+                    'menu_icon'    => 'dashicons-id',
+                ]
+            );
+
+            register_post_type(
+                'strato_candidate',
+                [
+                    'labels'       => [
+                        'name'               => __( 'Kandidaten', 'strato-job-manager' ),
+                        'singular_name'      => __( 'Kandidaat', 'strato-job-manager' ),
+                        'add_new_item'       => __( 'Nieuwe kandidaat', 'strato-job-manager' ),
+                        'edit_item'          => __( 'Kandidaat bewerken', 'strato-job-manager' ),
+                        'view_item'          => __( 'Kandidaat bekijken', 'strato-job-manager' ),
+                        'search_items'       => __( 'Kandidaten zoeken', 'strato-job-manager' ),
+                        'not_found'          => __( 'Geen kandidaten gevonden', 'strato-job-manager' ),
+                        'menu_name'          => __( 'Kandidaten', 'strato-job-manager' ),
+                    ],
+                    'public'       => false,
+                    'show_ui'      => true,
+                    'show_in_menu' => true,
+                    'supports'     => [ 'title', 'editor', 'author' ],
+                    'menu_icon'    => 'dashicons-groups',
+                ]
+            );
+        }
+
+        /**
+         * Register shortcodes.
+         */
+        public function register_shortcodes() {
+            add_shortcode( 'strato_vacancies', [ $this, 'render_vacancy_list' ] );
+        }
+
+        /**
+         * Register meta boxes for custom post types.
+         */
+        public function register_meta_boxes() {
+            add_meta_box(
+                'strato_vacancy_details',
+                __( 'Vacature details', 'strato-job-manager' ),
+                [ $this, 'render_vacancy_meta_box' ],
+                'strato_vacancy',
+                'normal',
+                'high'
+            );
+
+            add_meta_box(
+                'strato_candidate_details',
+                __( 'Kandidaat details', 'strato-job-manager' ),
+                [ $this, 'render_candidate_meta_box' ],
+                'strato_candidate',
+                'normal',
+                'high'
+            );
+        }
+
+        /**
+         * Render vacancy meta box.
+         */
+        public function render_vacancy_meta_box( $post ) {
+            wp_nonce_field( 'strato_save_vacancy', 'strato_vacancy_nonce' );
+
+            echo '<table class="form-table strato-vacancy-table">';
+            foreach ( $this->vacancy_fields as $key => $field ) {
+                $value = get_post_meta( $post->ID, $key, true );
+                echo '<tr>';
+                printf( '<th scope="row"><label for="%1$s">%2$s</label></th>', esc_attr( $key ), esc_html( $field['label'] ) );
+                echo '<td>';
+                switch ( $field['type'] ) {
+                    case 'textarea':
+                        printf(
+                            '<textarea id="%1$s" name="%1$s" rows="4" class="widefat">%2$s</textarea>',
+                            esc_attr( $key ),
+                            esc_textarea( $value )
+                        );
+                        break;
+                    case 'date':
+                        printf(
+                            '<input type="date" id="%1$s" name="%1$s" value="%2$s" class="regular-text"/>',
+                            esc_attr( $key ),
+                            esc_attr( $value )
+                        );
+                        break;
+                    default:
+                        printf(
+                            '<input type="text" id="%1$s" name="%1$s" value="%2$s" class="regular-text"/>',
+                            esc_attr( $key ),
+                            esc_attr( $value )
+                        );
+                        break;
+                }
+                echo '</td>';
+                echo '</tr>';
+            }
+            echo '</table>';
+        }
+
+        /**
+         * Render candidate meta box.
+         */
+        public function render_candidate_meta_box( $post ) {
+            wp_nonce_field( 'strato_save_candidate', 'strato_candidate_nonce' );
+
+            echo '<table class="form-table strato-candidate-table">';
+            foreach ( $this->candidate_fields as $key => $field ) {
+                if ( 'strato_candidate_vacancy' === $key ) {
+                    $this->render_candidate_vacancy_field( $post );
+                    continue;
+                }
+
+                if ( 'file' === $field['type'] ) {
+                    $value    = get_post_meta( $post->ID, $key, true );
+                    $filename = $value ? basename( get_attached_file( (int) $value ) ) : '';
+                    echo '<tr>';
+                    printf( '<th scope="row">%s</th>', esc_html( $field['label'] ) );
+                    echo '<td>';
+                    if ( $value ) {
+                        printf( '<a href="%1$s" target="_blank" rel="noopener">%2$s</a>', esc_url( wp_get_attachment_url( (int) $value ) ), esc_html( $filename ) );
+                    } else {
+                        esc_html_e( 'Geen bestand geüpload', 'strato-job-manager' );
+                    }
+                    echo '</td>';
+                    echo '</tr>';
+                    continue;
+                }
+
+                $value = get_post_meta( $post->ID, $key, true );
+                echo '<tr>';
+                printf( '<th scope="row"><label for="%1$s">%2$s</label></th>', esc_attr( $key ), esc_html( $field['label'] ) );
+                echo '<td>';
+                if ( 'textarea' === $field['type'] ) {
+                    printf(
+                        '<textarea id="%1$s" name="%1$s" rows="4" class="widefat">%2$s</textarea>',
+                        esc_attr( $key ),
+                        esc_textarea( $value )
+                    );
+                } else {
+                    printf(
+                        '<input type="text" id="%1$s" name="%1$s" value="%2$s" class="regular-text"/>',
+                        esc_attr( $key ),
+                        esc_attr( $value )
+                    );
+                }
+                echo '</td>';
+                echo '</tr>';
+            }
+            echo '</table>';
+        }
+
+        /**
+         * Render candidate vacancy selection field.
+         */
+        protected function render_candidate_vacancy_field( $post ) {
+            $value     = get_post_meta( $post->ID, 'strato_candidate_vacancy', true );
+            $vacancies = get_posts(
+                [
+                    'post_type'      => 'strato_vacancy',
+                    'posts_per_page' => -1,
+                    'post_status'    => 'publish',
+                    'orderby'        => 'title',
+                    'order'          => 'ASC',
+                ]
+            );
+
+            echo '<tr>';
+            printf( '<th scope="row"><label for="strato_candidate_vacancy">%s</label></th>', esc_html__( 'Vacature', 'strato-job-manager' ) );
+            echo '<td>';
+            echo '<select id="strato_candidate_vacancy" name="strato_candidate_vacancy" class="regular-text">';
+            echo '<option value="">' . esc_html__( 'Kies een vacature', 'strato-job-manager' ) . '</option>';
+            foreach ( $vacancies as $vacancy ) {
+                printf(
+                    '<option value="%1$s" %3$s>%2$s</option>',
+                    esc_attr( $vacancy->ID ),
+                    esc_html( $vacancy->post_title ),
+                    selected( (int) $value, $vacancy->ID, false )
+                );
+            }
+            echo '</select>';
+            echo '</td>';
+            echo '</tr>';
+        }
+
+        /**
+         * Save meta box data.
+         */
+        public function save_post_meta( $post_id ) {
+            if ( isset( $_POST['strato_vacancy_nonce'] ) && wp_verify_nonce( sanitize_text_field( wp_unslash( $_POST['strato_vacancy_nonce'] ) ), 'strato_save_vacancy' ) ) {
+                $this->save_fields( $post_id, $this->vacancy_fields );
+            }
+
+            if ( isset( $_POST['strato_candidate_nonce'] ) && wp_verify_nonce( sanitize_text_field( wp_unslash( $_POST['strato_candidate_nonce'] ) ), 'strato_save_candidate' ) ) {
+                $this->save_fields( $post_id, $this->candidate_fields );
+            }
+        }
+
+        /**
+         * Save array of fields as post meta.
+         *
+         * @param int   $post_id Post identifier.
+         * @param array $fields  Field definitions.
+         */
+        protected function save_fields( $post_id, $fields ) {
+            foreach ( $fields as $key => $field ) {
+                if ( 'file' === $field['type'] ) {
+                    continue;
+                }
+
+                $value = isset( $_POST[ $key ] ) ? wp_unslash( $_POST[ $key ] ) : '';
+
+                switch ( $field['type'] ) {
+                    case 'textarea':
+                        $sanitized = sanitize_textarea_field( $value );
+                        break;
+                    case 'email':
+                        $sanitized = sanitize_email( $value );
+                        break;
+                    case 'date':
+                        $sanitized = sanitize_text_field( $value );
+                        break;
+                    default:
+                        $sanitized = sanitize_text_field( $value );
+                        break;
+                }
+
+                if ( ! empty( $sanitized ) ) {
+                    update_post_meta( $post_id, $key, $sanitized );
+                } else {
+                    delete_post_meta( $post_id, $key );
+                }
+            }
+        }
+
+        /**
+         * Admin area assets.
+         */
+        public function admin_assets() {
+            $css = '.strato-vacancy-table th{width:180px;vertical-align:top;}'
+                . '.strato-candidate-table th{width:180px;vertical-align:top;}';
+
+            wp_register_style( 'strato-job-manager-admin', false );
+            wp_enqueue_style( 'strato-job-manager-admin' );
+            wp_add_inline_style( 'strato-job-manager-admin', $css );
+        }
+
+        /**
+         * Frontend assets.
+         */
+        public function frontend_assets() {
+            $css = '.strato-vacancies{display:grid;gap:2rem;margin:2rem 0;grid-template-columns:repeat(auto-fit,minmax(280px,1fr));}'
+                . '.strato-vacancy-card{border:1px solid #e2e8f0;border-radius:8px;padding:1.5rem;background:#fff;box-shadow:0 10px 30px rgba(15,23,42,0.08);}'
+                . '.strato-vacancy-card h3{margin-top:0;font-size:1.25rem;color:#0f172a;}'
+                . '.strato-vacancy-meta{list-style:none;margin:1rem 0;padding:0;}'
+                . '.strato-vacancy-meta li{margin-bottom:0.35rem;display:flex;justify-content:space-between;font-size:0.95rem;color:#334155;}'
+                . '.strato-vacancy-apply{display:inline-block;margin-top:1rem;padding:0.75rem 1.25rem;background:#2563eb;color:#fff;border-radius:999px;text-decoration:none;font-weight:600;}'
+                . '.strato-vacancy-apply:hover{background:#1d4ed8;}'
+                . '.strato-application-form{margin-top:2rem;border:1px solid #e2e8f0;border-radius:12px;padding:2rem;background:#f8fafc;}'
+                . '.strato-application-form .field{margin-bottom:1rem;display:flex;flex-direction:column;}'
+                . '.strato-application-form label{font-weight:600;margin-bottom:0.35rem;color:#0f172a;}'
+                . '.strato-application-form input[type="text"],.strato-application-form input[type="email"],.strato-application-form input[type="file"],.strato-application-form textarea{border:1px solid #cbd5f5;border-radius:8px;padding:0.65rem;font-size:1rem;}'
+                . '.strato-application-form textarea{min-height:120px;}'
+                . '.strato-application-form button{background:#2563eb;color:#fff;padding:0.85rem 1.5rem;border:none;border-radius:999px;font-weight:600;cursor:pointer;}'
+                . '.strato-application-form button:hover{background:#1d4ed8;}'
+                . '.strato-alert{padding:1rem 1.25rem;border-radius:8px;margin-bottom:1rem;}'
+                . '.strato-alert-error{background:#fee2e2;color:#991b1b;}'
+                . '.strato-alert-success{background:#dcfce7;color:#166534;}';
+
+            wp_register_style( 'strato-job-manager-frontend', false );
+            wp_enqueue_style( 'strato-job-manager-frontend' );
+            wp_add_inline_style( 'strato-job-manager-frontend', $css );
+        }
+
+        /**
+         * Append vacancy details and application form to the content.
+         *
+         * @param string $content Post content.
+         *
+         * @return string
+         */
+        public function append_vacancy_content( $content ) {
+            if ( ! is_singular( 'strato_vacancy' ) || ! in_the_loop() || ! is_main_query() ) {
+                return $content;
+            }
+
+            global $post;
+            $details_html = '<section class="strato-vacancy-details">';
+            $details_html .= '<h2>' . esc_html__( 'Functie-informatie', 'strato-job-manager' ) . '</h2>';
+            $details_html .= '<ul class="strato-vacancy-meta">';
+            foreach ( $this->vacancy_fields as $key => $field ) {
+                $value = get_post_meta( $post->ID, $key, true );
+                if ( empty( $value ) ) {
+                    continue;
+                }
+
+                $details_html .= sprintf(
+                    '<li><span>%1$s</span><span>%2$s</span></li>',
+                    esc_html( $field['label'] ),
+                    esc_html( $value )
+                );
+            }
+            $details_html .= '</ul>';
+            $details_html .= '</section>';
+
+            $form_html = $this->get_application_form( $post->ID );
+
+            return $content . $details_html . $form_html;
+        }
+
+        /**
+         * Generate application form HTML.
+         *
+         * @param int $vacancy_id Vacancy identifier.
+         */
+        protected function get_application_form( $vacancy_id ) {
+            $output = '<section class="strato-application-form">';
+            $output .= '<h2>' . esc_html__( 'Solliciteer op deze functie', 'strato-job-manager' ) . '</h2>';
+
+            if ( ! empty( $this->form_errors ) ) {
+                $output .= '<div class="strato-alert strato-alert-error">';
+                foreach ( $this->form_errors as $error ) {
+                    $output .= '<p>' . esc_html( $error ) . '</p>';
+                }
+                $output .= '</div>';
+            } elseif ( $this->form_success ) {
+                $output .= '<div class="strato-alert strato-alert-success">' . esc_html__( 'Bedankt voor je sollicitatie! We nemen zo snel mogelijk contact met je op.', 'strato-job-manager' ) . '</div>';
+            }
+
+            $output .= '<form method="post" enctype="multipart/form-data" class="strato-application" novalidate>';
+            $output .= wp_nonce_field( 'strato_submit_application', 'strato_application_nonce', true, false );
+            $output .= '<input type="hidden" name="strato_vacancy_id" value="' . esc_attr( $vacancy_id ) . '" />';
+            $output .= '<div class="field">';
+            $output .= '<label for="strato_applicant_name">' . esc_html__( 'Naam', 'strato-job-manager' ) . '</label>';
+            $output .= '<input type="text" id="strato_applicant_name" name="strato_applicant_name" required />';
+            $output .= '</div>';
+            $output .= '<div class="field">';
+            $output .= '<label for="strato_applicant_email">' . esc_html__( 'E-mailadres', 'strato-job-manager' ) . '</label>';
+            $output .= '<input type="email" id="strato_applicant_email" name="strato_applicant_email" required />';
+            $output .= '</div>';
+            $output .= '<div class="field">';
+            $output .= '<label for="strato_applicant_phone">' . esc_html__( 'Telefoonnummer', 'strato-job-manager' ) . '</label>';
+            $output .= '<input type="text" id="strato_applicant_phone" name="strato_applicant_phone" />';
+            $output .= '</div>';
+            $output .= '<div class="field">';
+            $output .= '<label for="strato_applicant_message">' . esc_html__( 'Motivatie', 'strato-job-manager' ) . '</label>';
+            $output .= '<textarea id="strato_applicant_message" name="strato_applicant_message" rows="5"></textarea>';
+            $output .= '</div>';
+            $output .= '<div class="field">';
+            $output .= '<label for="strato_applicant_cv">' . esc_html__( 'CV uploaden (PDF, DOC, DOCX)', 'strato-job-manager' ) . '</label>';
+            $output .= '<input type="file" id="strato_applicant_cv" name="strato_applicant_cv" accept=".pdf,.doc,.docx" />';
+            $output .= '</div>';
+            $output .= '<div class="field">';
+            $output .= '<label>';
+            $output .= '<input type="checkbox" name="strato_applicant_consent" value="1" required /> ';
+            $output .= esc_html__( 'Ik ga akkoord met het verwerken van mijn gegevens voor deze sollicitatie.', 'strato-job-manager' );
+            $output .= '</label>';
+            $output .= '</div>';
+            $output .= '<button type="submit" name="strato_job_application" value="1">' . esc_html__( 'Solliciteren', 'strato-job-manager' ) . '</button>';
+            $output .= '</form>';
+            $output .= '</section>';
+
+            return $output;
+        }
+
+        /**
+         * Handle application form submission.
+         */
+        public function handle_application_form() {
+            if ( ! isset( $_POST['strato_job_application'] ) ) {
+                if ( isset( $_GET['strato_application_success'] ) ) {
+                    $this->form_success = (bool) intval( $_GET['strato_application_success'] );
+                }
+                return;
+            }
+
+            if ( ! isset( $_POST['strato_application_nonce'] ) || ! wp_verify_nonce( sanitize_text_field( wp_unslash( $_POST['strato_application_nonce'] ) ), 'strato_submit_application' ) ) {
+                $this->form_errors[] = __( 'Ongeldige aanvraag, probeer het opnieuw.', 'strato-job-manager' );
+                return;
+            }
+
+            $name    = isset( $_POST['strato_applicant_name'] ) ? sanitize_text_field( wp_unslash( $_POST['strato_applicant_name'] ) ) : '';
+            $email   = isset( $_POST['strato_applicant_email'] ) ? sanitize_email( wp_unslash( $_POST['strato_applicant_email'] ) ) : '';
+            $phone   = isset( $_POST['strato_applicant_phone'] ) ? sanitize_text_field( wp_unslash( $_POST['strato_applicant_phone'] ) ) : '';
+            $message = isset( $_POST['strato_applicant_message'] ) ? sanitize_textarea_field( wp_unslash( $_POST['strato_applicant_message'] ) ) : '';
+            $consent = isset( $_POST['strato_applicant_consent'] );
+            $vacancy = isset( $_POST['strato_vacancy_id'] ) ? absint( $_POST['strato_vacancy_id'] ) : 0;
+
+            if ( empty( $name ) ) {
+                $this->form_errors[] = __( 'Naam is verplicht.', 'strato-job-manager' );
+            }
+
+            if ( empty( $email ) || ! is_email( $email ) ) {
+                $this->form_errors[] = __( 'Een geldig e-mailadres is verplicht.', 'strato-job-manager' );
+            }
+
+            if ( ! $consent ) {
+                $this->form_errors[] = __( 'Je moet toestemming geven voor het verwerken van je gegevens.', 'strato-job-manager' );
+            }
+
+            if ( empty( $vacancy ) || 'strato_vacancy' !== get_post_type( $vacancy ) ) {
+                $this->form_errors[] = __( 'De geselecteerde vacature bestaat niet meer.', 'strato-job-manager' );
+            }
+
+            $attachment_id = null;
+            if ( isset( $_FILES['strato_applicant_cv'] ) && ! empty( $_FILES['strato_applicant_cv']['name'] ) ) {
+                require_once ABSPATH . 'wp-admin/includes/file.php';
+                $file = wp_handle_upload( $_FILES['strato_applicant_cv'], [ 'test_form' => false ] );
+                if ( isset( $file['error'] ) ) {
+                    $this->form_errors[] = $file['error'];
+                } else {
+                    $attachment = [
+                        'post_title'     => $name . ' CV',
+                        'post_content'   => '',
+                        'post_status'    => 'private',
+                        'post_mime_type' => $file['type'],
+                        'guid'           => $file['url'],
+                    ];
+                    $attachment_id = wp_insert_attachment( $attachment, $file['file'] );
+                    if ( ! is_wp_error( $attachment_id ) ) {
+                        require_once ABSPATH . 'wp-admin/includes/image.php';
+                        if ( wp_attachment_is_image( $attachment_id ) ) {
+                            wp_update_attachment_metadata( $attachment_id, wp_generate_attachment_metadata( $attachment_id, $file['file'] ) );
+                        }
+                    }
+                }
+            }
+
+            if ( ! empty( $this->form_errors ) ) {
+                return;
+            }
+
+            $candidate_id = wp_insert_post(
+                [
+                    'post_type'   => 'strato_candidate',
+                    'post_status' => 'private',
+                    'post_title'  => sprintf( __( '%1$s - Sollicitatie voor %2$s', 'strato-job-manager' ), $name, get_the_title( $vacancy ) ),
+                    'post_content'=> $message,
+                ],
+                true
+            );
+
+            if ( is_wp_error( $candidate_id ) ) {
+                $this->form_errors[] = __( 'Er is iets misgegaan bij het opslaan van je sollicitatie. Probeer het later opnieuw.', 'strato-job-manager' );
+                return;
+            }
+
+            update_post_meta( $candidate_id, 'strato_candidate_email', $email );
+            update_post_meta( $candidate_id, 'strato_candidate_phone', $phone );
+            update_post_meta( $candidate_id, 'strato_candidate_vacancy', $vacancy );
+
+            if ( $attachment_id ) {
+                update_post_meta( $candidate_id, 'strato_candidate_cv', $attachment_id );
+            }
+
+            $admin_email = get_option( 'admin_email' );
+            $subject     = sprintf( __( 'Nieuwe sollicitatie voor %s', 'strato-job-manager' ), get_the_title( $vacancy ) );
+            $body        = sprintf(
+                "%1\$s\n\nVacature: %2\$s\nE-mailadres: %3\$s\nTelefoonnummer: %4\$s\n\nMotivatie:\n%5\$s",
+                $name,
+                get_the_title( $vacancy ),
+                $email,
+                $phone,
+                $message
+            );
+
+            wp_mail( $admin_email, $subject, $body );
+
+            $redirect = add_query_arg( [ 'strato_application_success' => 1 ], get_permalink( $vacancy ) );
+            wp_safe_redirect( $redirect );
+            exit;
+        }
+
+        /**
+         * Render vacancy list shortcode.
+         */
+        public function render_vacancy_list( $atts ) {
+            $atts = shortcode_atts(
+                [
+                    'posts_per_page' => 12,
+                    'order'          => 'DESC',
+                    'orderby'        => 'date',
+                ],
+                $atts,
+                'strato_vacancies'
+            );
+
+            $query = new WP_Query(
+                [
+                    'post_type'      => 'strato_vacancy',
+                    'post_status'    => 'publish',
+                    'posts_per_page' => absint( $atts['posts_per_page'] ),
+                    'orderby'        => sanitize_key( $atts['orderby'] ),
+                    'order'          => 'ASC' === strtoupper( $atts['order'] ) ? 'ASC' : 'DESC',
+                ]
+            );
+
+            if ( ! $query->have_posts() ) {
+                return '<p>' . esc_html__( 'Momenteel zijn er geen vacatures beschikbaar.', 'strato-job-manager' ) . '</p>';
+            }
+
+            ob_start();
+            echo '<div class="strato-vacancies">';
+            while ( $query->have_posts() ) {
+                $query->the_post();
+                $meta = [];
+                foreach ( $this->vacancy_fields as $key => $field ) {
+                    $value = get_post_meta( get_the_ID(), $key, true );
+                    if ( empty( $value ) ) {
+                        continue;
+                    }
+
+                    $meta[] = sprintf(
+                        '<li><span>%1$s</span><span>%2$s</span></li>',
+                        esc_html( $field['label'] ),
+                        esc_html( $value )
+                    );
+                }
+
+                echo '<article class="strato-vacancy-card">';
+                echo '<h3>' . esc_html( get_the_title() ) . '</h3>';
+                if ( has_post_thumbnail() ) {
+                    echo '<div class="strato-vacancy-thumb">' . get_the_post_thumbnail( get_the_ID(), 'medium_large' ) . '</div>';
+                }
+                echo '<div class="strato-vacancy-excerpt">' . wp_kses_post( wpautop( get_the_excerpt() ) ) . '</div>';
+                if ( ! empty( $meta ) ) {
+                    echo '<ul class="strato-vacancy-meta">' . implode( '', $meta ) . '</ul>';
+                }
+                echo '<a class="strato-vacancy-apply" href="' . esc_url( get_permalink() ) . '">' . esc_html__( 'Bekijk vacature', 'strato-job-manager' ) . '</a>';
+                echo '</article>';
+            }
+            echo '</div>';
+            wp_reset_postdata();
+
+            return ob_get_clean();
+        }
+
+        /**
+         * Register admin columns for vacancies.
+         */
+        public function register_vacancy_columns( $columns ) {
+            $columns['strato_vacancy_location'] = __( 'Locatie', 'strato-job-manager' );
+            $columns['strato_vacancy_hourly']   = __( 'Uurloon', 'strato-job-manager' );
+            $columns['strato_vacancy_monthly']  = __( 'Maandsalaris', 'strato-job-manager' );
+            $columns['strato_vacancy_contract'] = __( 'Contractduur', 'strato-job-manager' );
+            return $columns;
+        }
+
+        /**
+         * Render vacancy column values.
+         */
+        public function render_vacancy_columns( $column, $post_id ) {
+            $value = get_post_meta( $post_id, $column, true );
+            if ( $value ) {
+                echo esc_html( $value );
+            }
+        }
+
+        /**
+         * Register admin columns for candidates.
+         */
+        public function register_candidate_columns( $columns ) {
+            $new_columns = [];
+            foreach ( $columns as $key => $label ) {
+                $new_columns[ $key ] = $label;
+                if ( 'title' === $key ) {
+                    $new_columns['candidate_vacancy'] = __( 'Vacature', 'strato-job-manager' );
+                    $new_columns['candidate_email']   = __( 'E-mailadres', 'strato-job-manager' );
+                    $new_columns['candidate_phone']   = __( 'Telefoonnummer', 'strato-job-manager' );
+                }
+            }
+
+            return $new_columns;
+        }
+
+        /**
+         * Render candidate column values.
+         */
+        public function render_candidate_columns( $column, $post_id ) {
+            switch ( $column ) {
+                case 'candidate_vacancy':
+                    $vacancy_id = get_post_meta( $post_id, 'strato_candidate_vacancy', true );
+                    if ( $vacancy_id ) {
+                        printf( '<a href="%1$s">%2$s</a>', esc_url( get_edit_post_link( $vacancy_id ) ), esc_html( get_the_title( $vacancy_id ) ) );
+                    }
+                    break;
+                case 'candidate_email':
+                    $email = get_post_meta( $post_id, 'strato_candidate_email', true );
+                    if ( $email ) {
+                        printf( '<a href="mailto:%1$s">%1$s</a>', esc_html( $email ) );
+                    }
+                    break;
+                case 'candidate_phone':
+                    $phone = get_post_meta( $post_id, 'strato_candidate_phone', true );
+                    if ( $phone ) {
+                        echo esc_html( $phone );
+                    }
+                    break;
+            }
+        }
+
+        /**
+         * Add filters to admin list tables.
+         */
+        public function admin_filters() {
+            global $typenow;
+
+            if ( 'strato_vacancy' === $typenow ) {
+                $selected = isset( $_GET['strato_vacancy_location'] ) ? sanitize_text_field( wp_unslash( $_GET['strato_vacancy_location'] ) ) : '';
+                echo '<input type="search" placeholder="' . esc_attr__( 'Filter op locatie', 'strato-job-manager' ) . '" name="strato_vacancy_location" value="' . esc_attr( $selected ) . '" />';
+            }
+
+            if ( 'strato_candidate' === $typenow ) {
+                $vacancies = get_posts(
+                    [
+                        'post_type'      => 'strato_vacancy',
+                        'post_status'    => 'publish',
+                        'posts_per_page' => -1,
+                        'orderby'        => 'title',
+                        'order'          => 'ASC',
+                    ]
+                );
+                $selected = isset( $_GET['strato_candidate_vacancy'] ) ? absint( $_GET['strato_candidate_vacancy'] ) : '';
+                echo '<select name="strato_candidate_vacancy">';
+                echo '<option value="">' . esc_html__( 'Alle vacatures', 'strato-job-manager' ) . '</option>';
+                foreach ( $vacancies as $vacancy ) {
+                    printf(
+                        '<option value="%1$s" %3$s>%2$s</option>',
+                        esc_attr( $vacancy->ID ),
+                        esc_html( $vacancy->post_title ),
+                        selected( $selected, $vacancy->ID, false )
+                    );
+                }
+                echo '</select>';
+            }
+        }
+
+        /**
+         * Apply admin filters to queries.
+         */
+        public function apply_admin_filters( $query ) {
+            global $pagenow;
+            if ( ! is_admin() || 'edit.php' !== $pagenow || ! $query->is_main_query() ) {
+                return;
+            }
+
+            $post_type = isset( $_GET['post_type'] ) ? sanitize_key( $_GET['post_type'] ) : 'post';
+
+            if ( 'strato_vacancy' === $post_type && ! empty( $_GET['strato_vacancy_location'] ) ) {
+                $location = sanitize_text_field( wp_unslash( $_GET['strato_vacancy_location'] ) );
+                $meta_query = [
+                    [
+                        'key'     => 'strato_vacancy_location',
+                        'value'   => $location,
+                        'compare' => 'LIKE',
+                    ],
+                ];
+                $query->set( 'meta_query', $meta_query );
+            }
+
+            if ( 'strato_candidate' === $post_type && ! empty( $_GET['strato_candidate_vacancy'] ) ) {
+                $query->set(
+                    'meta_query',
+                    [
+                        [
+                            'key'   => 'strato_candidate_vacancy',
+                            'value' => absint( $_GET['strato_candidate_vacancy'] ),
+                        ],
+                    ]
+                );
+            }
+        }
+    }
+}
+
+Strato_Job_Manager::instance();
