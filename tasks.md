# Implementation Plan: BharatChef AI Coach

## Overview

This implementation plan breaks down the BharatChef AI Coach platform into discrete coding tasks. The system is a cloud-based cooking skill evaluation platform using computer vision, temporal analysis, and agentic AI coaching. The implementation uses Python for backend services, AWS cloud infrastructure, and integrates multiple CV models (SlowFast/I3D, YOLOv8, ResNet) with multimodal LLM-powered coaching.

The tasks are organized to build incrementally: infrastructure setup → core CV modules → analysis layer → AI coaching → API layer → testing and integration.

## Tasks

- [x] 1. Set up project infrastructure and core data models
  - Create Python project structure with FastAPI, Celery, PostgreSQL, Redis
  - Define SQLAlchemy models for Video, Dish, Evaluation, Trainee entities
  - Set up AWS SDK configuration (S3, Lambda, SQS, Bedrock)
  - Create database migration scripts
  - Set up environment configuration management
  - _Requirements: 1.1, 1.3, 1.4, 2.1, 2.2, 14.1, 14.2_

- [ ]* 1.1 Write property test for data model round-trip persistence
  - **Property 1: Video Upload and Retrieval Round Trip**
  - **Validates: Requirements 1.3, 1.4, 1.6, 1.8, 1.9**

- [ ] 2. Implement Video Capture and Upload Service
  - [ ] 2.1 Create video upload API endpoint with multipart file handling
    - Implement FastAPI endpoint for video upload with progress tracking
    - Add video format validation (MP4, MOV, AVI)
    - Add duration validation (30 seconds to 30 minutes)
    - Add file size validation
    - _Requirements: 1.1, 1.3, 1.8, 1.9_
  
  - [ ] 2.2 Implement S3 upload with resumable upload support
    - Use boto3 multipart upload for large files
    - Store upload checkpoints in Redis for resumption
    - Handle network interruption and resume logic
    - Generate unique video IDs and cloud URLs
    - _Requirements: 1.4, 1.6, 1.7_
  
  - [ ]* 2.3 Write property test for upload resumption
    - **Property 2: Upload Resumption After Network Interruption**
    - **Validates: Requirements 1.7**
  
  - [ ]* 2.4 Write unit tests for video validation
    - Test valid formats (MP4, MOV, AVI)
    - Test invalid formats rejection
    - Test duration edge cases (30s minimum, 30min maximum)
    - Test file size limits
    - _Requirements: 1.8, 1.9_

- [ ] 3. Implement Dish Metadata Management Service
  - [ ] 3.1 Create dish CRUD API endpoints
    - Implement create dish endpoint with metadata validation
    - Implement get dish endpoint with expert video association
    - Implement list dishes endpoint
    - Store dish metadata in PostgreSQL
    - _Requirements: 2.1, 2.4, 2.5_
  
  - [ ] 3.2 Implement expert video association logic
    - Create endpoint to associate expert video with dish
    - Validate expert video contains expected cooking steps
    - Store association in database
    - _Requirements: 2.2, 2.3_
  
  - [ ]* 3.3 Write property test for dish metadata association
    - **Property 3: Dish Metadata Association Preservation**
    - **Validates: Requirements 2.2, 2.4**
  
  - [ ]* 3.4 Write unit tests for dish validation
    - Test dish creation with valid metadata
    - Test expert video validation logic
    - Test missing expert video error handling
    - _Requirements: 2.3, 13.4_

- [ ] 4. Checkpoint - Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 5. Implement Action Recognition Module
  - [ ] 5.1 Set up SlowFast or I3D model inference pipeline
    - Load pre-trained SlowFast/I3D model using PyTorch
    - Implement video frame extraction at 8-16 FPS
    - Create batch inference logic for efficiency
    - Configure GPU inference on AWS EC2 G4 instances
    - _Requirements: 3.1_
  
  - [ ] 5.2 Implement action detection and sequence generation
    - Process video frames through action recognition model
    - Detect cooking actions (chopping, stirring, flipping, pouring, plating)
    - Record action type, start/end timestamps, confidence scores
    - Generate structured action sequence with temporal ordering
    - Store results in database
    - _Requirements: 3.2, 3.3, 3.5_
  
  - [ ]* 5.3 Write property test for action detection completeness
    - **Property 4: Action Detection Completeness**
    - **Validates: Requirements 3.3**
  
  - [ ]* 5.4 Write property test for action sequence temporal ordering
    - **Property 5: Action Sequence Temporal Ordering**
    - **Validates: Requirements 3.5**
  
  - [ ]* 5.5 Write unit tests for action recognition
    - Test action detection with sample video frames
    - Test confidence score filtering (≥85% accuracy target)
    - Test temporal ordering of detected actions
    - _Requirements: 3.4_

- [ ] 6. Implement Object and Ingredient Detection Module
  - [ ] 6.1 Set up YOLOv8 model inference pipeline
    - Load YOLOv8 model fine-tuned on cooking dataset
    - Implement frame sampling at 1-2 FPS for ingredient detection
    - Create batch inference logic
    - Configure GPU inference
    - _Requirements: 4.1_
  
  - [ ] 6.2 Implement ingredient and utensil detection
    - Detect ingredients (vegetables, proteins, spices, liquids)
    - Detect utensils (knives, pans, spatulas, bowls, plates)
    - Record object type, name, timestamp, bounding box, confidence
    - Aggregate detections across frames to build unique ingredient list
    - _Requirements: 4.2, 4.4, 4.5_
  
  - [ ] 6.3 Implement ingredient report generation
    - Compare detected ingredients with expected ingredients from dish metadata
    - Identify missing ingredients and extra ingredients
    - Calculate ingredient detection accuracy
    - Generate structured ingredient report
    - _Requirements: 4.6, 4.7_
  
  - [ ]* 6.4 Write property test for object detection completeness
    - **Property 6: Object Detection Completeness**
    - **Validates: Requirements 4.5**
  
  - [ ]* 6.5 Write property test for ingredient report accuracy
    - **Property 7: Ingredient Report Accuracy**
    - **Validates: Requirements 4.6, 4.7**
  
  - [ ]* 6.6 Write unit tests for object detection
    - Test ingredient detection with sample frames
    - Test utensil detection
    - Test ingredient report generation logic
    - Test accuracy target (≥80%)
    - _Requirements: 4.3_

- [ ] 7. Implement Heat and Flame Analysis Module
  - [ ] 7.1 Implement flame detection using brightness spectrum analysis
    - Extract frames from side camera video (if available)
    - Analyze brightness in HSV color space
    - Detect orange/yellow spectrum for flame presence
    - Classify flame level (low, medium, high, none) based on brightness
    - Record flame detections with timestamps
    - _Requirements: 5.1, 5.2, 5.3_
  
  - [ ] 7.2 Implement heat intensity estimation
    - Analyze visual cues (steam, bubbling, color changes)
    - Estimate heat intensity (low, medium, high)
    - Handle both side camera and overhead camera inputs
    - Record heat intensities with timestamps and confidence
    - _Requirements: 5.4, 5.7_
  
  - [ ] 7.3 Implement heat control scoring
    - Compare trainee heat patterns with expert heat patterns
    - Identify heat control deviations (overheating, underheating, timing)
    - Calculate heat control score (0-100)
    - _Requirements: 5.5, 5.6_
  
  - [ ]* 7.4 Write property test for flame classification consistency
    - **Property 8: Flame Classification Consistency**
    - **Validates: Requirements 5.3**
  
  - [ ]* 7.5 Write property test for heat control score generation
    - **Property 9: Heat Control Score Generation**
    - **Validates: Requirements 5.6, 5.7**
  
  - [ ]* 7.6 Write unit tests for flame analysis
    - Test flame detection with sample frames
    - Test heat intensity estimation
    - Test heat control scoring logic
    - _Requirements: 5.1, 5.2, 5.4_

- [ ] 8. Checkpoint - Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 9. Implement Temporal Alignment Module using DTW
  - [ ] 9.1 Implement Dynamic Time Warping algorithm
    - Create DTW implementation with cosine distance on action embeddings
    - Calculate cost matrix and optimal alignment path
    - Return alignment distance and path
    - _Requirements: 6.1_
  
  - [ ] 9.2 Implement sequence alignment and deviation detection
    - Align trainee action sequence with expert action sequence
    - Identify missing steps, extra steps, out-of-order steps
    - Calculate timing differences for aligned steps
    - Calculate step sequence accuracy percentage
    - Generate step deviation report with impact levels
    - _Requirements: 6.2, 6.3, 6.4, 6.5_
  
  - [ ]* 9.3 Write property test for DTW alignment symmetry
    - **Property 10: DTW Alignment Symmetry**
    - **Validates: Requirements 6.1**
  
  - [ ]* 9.4 Write property test for step deviation identification
    - **Property 11: Step Deviation Identification**
    - **Validates: Requirements 6.2**
  
  - [ ]* 9.5 Write unit tests for temporal alignment
    - Test DTW with sample action sequences
    - Test missing step detection
    - Test extra step detection
    - Test out-of-order step detection
    - Test timing difference calculation
    - _Requirements: 6.1, 6.2, 6.3_

- [ ] 10. Implement Visual Quality Analysis Module
  - [ ] 10.1 Set up ResNet-50 model for visual embeddings
    - Load pre-trained ResNet-50 model from PyTorch
    - Implement frame extraction for final plating frames
    - Create embedding extraction logic
    - _Requirements: 7.1_
  
  - [ ] 10.2 Implement visual similarity calculation
    - Extract embeddings from trainee and expert final frames
    - Calculate cosine similarity between embeddings
    - Analyze color histograms and distributions
    - Analyze texture patterns
    - Identify specific visual differences (color, texture, plating)
    - Generate visual similarity score (0-100)
    - _Requirements: 7.2, 7.3, 7.4, 7.5_
  
  - [ ]* 10.3 Write property test for visual similarity score bounds
    - **Property 12: Visual Similarity Score Bounds**
    - **Validates: Requirements 7.2, 7.4**
  
  - [ ]* 10.4 Write unit tests for visual analysis
    - Test embedding extraction
    - Test cosine similarity calculation
    - Test color distribution analysis
    - Test texture pattern analysis
    - Test identical frames produce similarity of 1.0
    - _Requirements: 7.1, 7.2, 7.3_

- [ ] 11. Implement Scoring Engine
  - [ ] 11.1 Implement category score calculation
    - Calculate step sequence accuracy score
    - Calculate timing accuracy score
    - Calculate technique quality score
    - Calculate heat control score
    - Calculate visual quality score
    - Calculate plating score
    - Normalize all scores to 0-100 scale
    - _Requirements: 8.2_
  
  - [ ] 11.2 Implement weighted aggregation for overall score
    - Apply weights: step sequence (25%), timing (20%), technique (20%), visual quality (20%), heat control (10%), plating (5%)
    - Calculate overall performance score (0-100)
    - _Requirements: 8.1, 8.3, 8.4_
  
  - [ ] 11.3 Implement skill level determination
    - Determine skill level based on overall score
    - Beginner: 0-50, Intermediate: 51-75, Advanced: 76-100
    - _Requirements: 8.6_
  
  - [ ] 11.4 Implement score persistence with metadata
    - Store performance scores in database with timestamp and video ID
    - Ensure retrieval preserves all metadata
    - _Requirements: 8.5_
  
  - [ ]* 11.5 Write property test for weighted aggregation
    - **Property 13: Performance Score Weighted Aggregation**
    - **Validates: Requirements 8.1, 8.3, 8.4**
  
  - [ ]* 11.6 Write property test for skill level determination
    - **Property 14: Skill Level Determination**
    - **Validates: Requirements 8.6**
  
  - [ ]* 11.7 Write property test for score persistence
    - **Property 15: Score Persistence with Metadata**
    - **Validates: Requirements 8.5**
  
  - [ ]* 11.8 Write unit tests for scoring engine
    - Test category score calculations
    - Test weighted aggregation with known inputs
    - Test skill level boundaries
    - Test score storage and retrieval
    - _Requirements: 8.1, 8.2, 8.3, 8.4, 8.5, 8.6_

- [ ] 12. Checkpoint - Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 13. Implement AI Coach Agent with Multimodal LLM
  - [ ] 13.1 Set up AWS Bedrock integration for multimodal LLM
    - Configure AWS Bedrock client (Claude 3 or GPT-4V)
    - Implement LLM API call with retry logic and exponential backoff
    - Handle LLM timeouts with fallback to rule-based feedback
    - _Requirements: 9.1, 9.2_
  
  - [ ] 13.2 Implement CV output interpretation logic
    - Aggregate CV outputs (actions, objects, flames, visual, deviations)
    - Structure data for LLM prompt
    - Identify critical mistakes from CV data
    - Categorize mistakes (sequence, timing, technique, heat, visual, ingredient)
    - Calculate mistake severity and impact on score
    - _Requirements: 9.1, 9.2_
  
  - [ ] 13.3 Implement mistake prioritization
    - Rank mistakes by impact on overall performance
    - Select top 3-5 most critical mistakes
    - Generate reasoning for prioritization
    - _Requirements: 9.3, 9.6_
  
  - [ ] 13.4 Implement coaching feedback generation
    - Create structured prompt with CV data, scores, video frames
    - Request LLM to generate human-like feedback
    - Parse LLM response into structured feedback format
    - Generate mistake descriptions, impact explanations, corrective actions
    - Include encouraging language and practice recommendations
    - Add timestamps for video reference
    - _Requirements: 9.4, 9.5, 9.7, 9.8_
  
  - [ ]* 13.5 Write property test for feedback completeness
    - **Property 16: AI Coach Feedback Completeness**
    - **Validates: Requirements 9.3, 9.5, 9.8**
  
  - [ ]* 13.6 Write property test for feedback prioritization
    - **Property 17: Feedback Prioritization by Impact**
    - **Validates: Requirements 9.6**
  
  - [ ]* 13.7 Write unit tests for AI coach
    - Test CV output interpretation with mock data
    - Test mistake prioritization logic
    - Test LLM integration with mock responses
    - Test fallback feedback generation
    - Test feedback structure validation
    - _Requirements: 9.1, 9.2, 9.3, 9.4, 9.5, 9.6, 9.7, 9.8_

- [ ] 14. Implement Skill Progression Tracking Service
  - [ ] 14.1 Implement multi-attempt score storage and retrieval
    - Store all performance scores for each trainee-dish combination
    - Retrieve attempt history in chronological order
    - _Requirements: 11.1_
  
  - [ ] 14.2 Implement improvement calculation
    - Calculate improvement percentage between attempts
    - Calculate overall skill improvement trend
    - Determine skill trend (improving, stable, declining)
    - _Requirements: 11.2, 11.6_
  
  - [ ] 14.3 Implement repeated mistake detection
    - Identify mistakes that appear in consecutive attempts
    - Track mistake reduction percentage across attempts
    - Flag repeated mistakes for emphasis in feedback
    - _Requirements: 11.3, 11.4, 11.5_
  
  - [ ]* 14.4 Write property test for multi-attempt score storage
    - **Property 18: Multi-Attempt Score Storage**
    - **Validates: Requirements 11.1**
  
  - [ ]* 14.5 Write property test for skill improvement calculation
    - **Property 19: Skill Improvement Calculation**
    - **Validates: Requirements 11.2, 11.6**
  
  - [ ]* 14.6 Write property test for repeated mistake identification
    - **Property 20: Repeated Mistake Identification**
    - **Validates: Requirements 11.3, 11.4**
  
  - [ ]* 14.7 Write unit tests for progression tracking
    - Test score storage for multiple attempts
    - Test improvement percentage calculation
    - Test repeated mistake detection logic
    - Test skill trend determination
    - _Requirements: 11.1, 11.2, 11.3, 11.4, 11.5, 11.6_

- [ ] 15. Implement Results Dashboard API
  - [ ] 15.1 Create evaluation results API endpoint
    - Implement endpoint to retrieve complete evaluation results
    - Return performance score, coaching feedback, step deviations, visual comparison, ingredient report
    - _Requirements: 10.1, 10.2, 10.3, 10.4, 10.5_
  
  - [ ] 15.2 Create progression data API endpoint
    - Implement endpoint to retrieve skill progression data
    - Return attempt history, improvement percentage, repeated mistakes, skill trend
    - Generate progression chart data
    - _Requirements: 10.6_
  
  - [ ] 15.3 Create video comparison API endpoint
    - Implement endpoint for side-by-side video comparison
    - Return trainee and expert video URLs with synced timestamps
    - Generate sync points based on DTW alignment
    - _Requirements: 10.7_
  
  - [ ]* 15.4 Write unit tests for dashboard API
    - Test evaluation results endpoint
    - Test progression data endpoint
    - Test video comparison endpoint
    - Test data authorization (trainee can only access own data)
    - _Requirements: 10.1, 10.2, 10.3, 10.4, 10.5, 10.6, 10.7, 14.3_

- [ ] 16. Implement Processing Orchestration and Queue Management
  - [ ] 16.1 Create Celery task for video processing pipeline
    - Define Celery task that orchestrates full CV pipeline
    - Chain action recognition → object detection → flame analysis → visual analysis → temporal alignment → scoring → AI coaching
    - Handle task failures with retry logic
    - Update processing status in database
    - _Requirements: 12.1, 12.2_
  
  - [ ] 16.2 Implement SQS queue integration
    - Configure SQS queue for video processing jobs
    - Implement job submission on video upload
    - Implement worker polling and job processing
    - Handle concurrent job processing without blocking
    - _Requirements: 12.4_
  
  - [ ]* 16.3 Write property test for concurrent processing
    - **Property 21: Concurrent Video Processing Queue**
    - **Validates: Requirements 12.4**
  
  - [ ]* 16.4 Write unit tests for orchestration
    - Test pipeline task execution
    - Test error handling and retries
    - Test status updates
    - Test queue submission and processing
    - _Requirements: 12.1, 12.2, 12.4_

- [ ] 17. Checkpoint - Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 18. Implement Error Handling and Validation
  - [ ] 18.1 Implement video validation with specific error messages
    - Validate video format, duration, file size before processing
    - Return specific error messages for each validation failure
    - _Requirements: 13.6, 13.7_
  
  - [ ] 18.2 Implement low-confidence detection flagging
    - Flag action detections with confidence <60%
    - Include flags in evaluation report
    - _Requirements: 13.2_
  
  - [ ] 18.3 Implement dish-video mismatch detection
    - Validate trainee video matches selected dish
    - Notify trainee if mismatch detected
    - _Requirements: 13.3_
  
  - [ ] 18.4 Implement missing expert video validation
    - Check for expert video before evaluation
    - Prevent evaluation and notify administrator if missing
    - _Requirements: 13.4_
  
  - [ ] 18.5 Implement cloud processing error handling
    - Log processing errors with context
    - Notify trainee on failure
    - Provide retry option
    - _Requirements: 13.5_
  
  - [ ] 18.6 Implement upload failure handling
    - Display clear error messages on upload failure
    - Allow retry with preserved state
    - _Requirements: 13.1_
  
  - [ ]* 18.7 Write property test for low confidence flagging
    - **Property 22: Low Confidence Detection Flagging**
    - **Validates: Requirements 13.2**
  
  - [ ]* 18.8 Write property test for missing expert video prevention
    - **Property 23: Missing Expert Video Evaluation Prevention**
    - **Validates: Requirements 13.4**
  
  - [ ]* 18.9 Write property test for validation error specificity
    - **Property 24: Video Validation Error Specificity**
    - **Validates: Requirements 13.6, 13.7**
  
  - [ ]* 18.10 Write unit tests for error handling
    - Test video validation errors
    - Test low-confidence flagging
    - Test dish-video mismatch detection
    - Test missing expert video handling
    - Test cloud processing error handling
    - Test upload failure handling
    - _Requirements: 13.1, 13.2, 13.3, 13.4, 13.5, 13.6, 13.7_

- [ ] 19. Implement Security and Data Privacy Features
  - [ ] 19.1 Implement TLS encryption for video uploads
    - Configure HTTPS endpoints with TLS certificates
    - Enforce encrypted transmission for all video uploads
    - _Requirements: 14.1_
  
  - [ ] 19.2 Implement secure video storage with access controls
    - Configure S3 bucket with IAM policies
    - Implement signed URLs for video access
    - Set up encryption at rest
    - _Requirements: 14.2_
  
  - [ ] 19.3 Implement trainee data access authorization
    - Add authentication middleware to API endpoints
    - Verify trainee can only access their own data
    - Return 403 Forbidden for unauthorized access
    - _Requirements: 14.3_
  
  - [ ] 19.4 Implement data deletion functionality
    - Create endpoint for trainee data deletion requests
    - Delete all associated videos and performance records
    - Complete deletion within 30 days
    - _Requirements: 14.4_
  
  - [ ]* 19.5 Write property test for data access authorization
    - **Property 25: Trainee Data Access Authorization**
    - **Validates: Requirements 14.3**
  
  - [ ]* 19.6 Write property test for data deletion completeness
    - **Property 26: Data Deletion Completeness**
    - **Validates: Requirements 14.4**
  
  - [ ]* 19.7 Write unit tests for security features
    - Test TLS encryption configuration
    - Test access control enforcement
    - Test data deletion logic
    - Test unauthorized access rejection
    - _Requirements: 14.1, 14.2, 14.3, 14.4, 14.5, 14.6_

- [ ] 20. Implement Performance Optimization
  - [ ] 20.1 Implement model quantization for CV models
    - Apply FP16 or INT8 quantization to SlowFast/I3D, YOLOv8, ResNet
    - Benchmark inference speed improvements
    - Validate accuracy remains within acceptable thresholds
    - _Requirements: 12.6_
  
  - [ ] 20.2 Implement batch processing for multiple videos
    - Group video processing jobs into batches
    - Process batches on GPU for efficiency
    - _Requirements: 12.6_
  
  - [ ] 20.3 Implement caching for expert video analysis
    - Cache expert video CV outputs (actions, objects, visual embeddings)
    - Reuse cached results for all trainee comparisons
    - Store cache in Redis with TTL
    - _Requirements: 12.3_
  
  - [ ]* 20.4 Write performance tests
    - Test processing time for 10-minute video (target: <5 minutes)
    - Test concurrent processing with 100 simultaneous uploads
    - Test cost per evaluation (target: <₹2)
    - _Requirements: 12.1, 12.3, 12.4_

- [ ] 21. Final Integration and End-to-End Testing
  - [ ] 21.1 Implement end-to-end evaluation flow
    - Wire all components together: upload → CV pipeline → scoring → AI coaching → results
    - Test complete flow with sample videos
    - Verify all data flows correctly through pipeline
    - _Requirements: All requirements_
  
  - [ ] 21.2 Implement multi-attempt progression flow
    - Test multiple attempts for same dish
    - Verify progression tracking works correctly
    - Verify repeated mistakes are identified and emphasized
    - _Requirements: 11.1, 11.2, 11.3, 11.4, 11.5, 11.6_
  
  - [ ] 21.3 Implement network interruption recovery flow
    - Test upload interruption and resumption
    - Verify no data loss or corruption
    - _Requirements: 1.7_
  
  - [ ]* 21.4 Write integration tests
    - Test end-to-end evaluation flow
    - Test multi-attempt progression flow
    - Test network interruption recovery
    - Test with real CV models and LLM
    - _Requirements: All requirements_

- [ ] 22. Final checkpoint - Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP
- Each task references specific requirements for traceability
- Property tests validate universal correctness properties (minimum 100 iterations each)
- Unit tests validate specific examples and edge cases
- The implementation uses Python with FastAPI, Celery, PostgreSQL, Redis, and AWS services
- CV models: SlowFast/I3D (action recognition), YOLOv8 (object detection), ResNet-50 (visual embeddings)
- AI Coach uses AWS Bedrock with Claude 3 or GPT-4V for multimodal LLM
- Target performance: <5 minutes processing for 10-minute video, <₹2 cost per evaluation
- Target accuracy: ≥85% action recognition, ≥80% ingredient detection
