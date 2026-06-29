
# Fastlane configuration for Flutter Android releases.
# Docs: https://docs.fastlane.tools/getting-started/android/release-deployment/

default_platform(:android)

def flutter_project_root
  File.expand_path("../..", __dir__)
end

def release_aab_path
  File.join(flutter_project_root, "build/app/outputs/bundle/release/app-release.aab")
end

def deploy_track
  ENV.fetch("PLAY_STORE_TRACK", "alpha")
end

platform :android do
  desc "Build a signed release app bundle with Flutter"
  lane :build do
    build_name = ENV.fetch("BUILD_NAME")
    build_number = ENV.fetch("BUILD_NUMBER")

    Dir.chdir(flutter_project_root) do
      # sh("flutter", "pub", "get")
      sh(
        "flutter", "build", "appbundle", "--release",
        "--build-name=#{build_name}",
        "--build-number=#{build_number}"
      )
    end

    UI.user_error!("AAB not found at #{release_aab_path}") unless File.exist?(release_aab_path)

    release_aab_path
  end

  desc "Deploy a new release to Google Play"
  lane :deploy do
    build_number = ENV.fetch("BUILD_NUMBER").to_i
    UI.user_error!("BUILD_NUMBER must be a positive integer from pubspec.yaml") if build_number <= 0
    UI.success("Using version code #{build_number} from pubspec.yaml")
    aab = build
    track = deploy_track

    upload_options = {
      aab: aab,
      track: track,
      json_key_data: ENV.fetch("PLAY_STORE_SERVICE_ACCOUNT_JSON"),
      skip_upload_apk: true,
      skip_upload_metadata: true,
      skip_upload_changelogs: true,
      skip_upload_images: true,
      skip_upload_screenshots: true
    }

    if track == "production"
      upload_options[:rollout] = "0.1"
      upload_options[:release_status] = "inProgress"
    else
      upload_options[:release_status] = "completed"
    end

    upload_to_play_store(upload_options)
  end
end
