package com.servlet;

import java.io.File;
import java.io.IOException;
import java.util.Iterator;
import java.util.List;

import javax.servlet.ServletConfig;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.tomcat.util.http.fileupload.FileItem;
import org.apache.tomcat.util.http.fileupload.FileItemFactory;
import org.apache.tomcat.util.http.fileupload.disk.DiskFileItemFactory;
import org.apache.tomcat.util.http.fileupload.servlet.ServletFileUpload;

import com.dto.GalleryDto;
import com.service.GalleryService;

/**
 * Servlet implementation class GalleryServlet
 */
public class GalleryServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;
	String path = "";
	String fullpath = "";
	String Name = "";

	/**
	 * @see HttpServlet#HttpServlet()
	 */
	public GalleryServlet() {
		super();
		// TODO Auto-generated constructor stub
	}

	public void init(ServletConfig config) throws ServletException {

		super.init();
		path = config.getServletContext().getRealPath("/");
		System.out.println(path);

	}

	/**
	 * @see HttpServlet#doGet(HttpServletRequest request, HttpServletResponse
	 *      response)
	 */
	protected void doGet(HttpServletRequest request,
			HttpServletResponse response) throws ServletException, IOException {
		// TODO Auto-generated method stub
	}

	/**
	 * @see HttpServlet#doPost(HttpServletRequest request, HttpServletResponse
	 *      response)
	 */
	protected void doPost(HttpServletRequest request,
			HttpServletResponse response) throws ServletException, IOException {
		String msg = "";
		File savedFile = null;
		FileItem item = null;
		GalleryDto dto = new GalleryDto();
		GalleryService service = new GalleryService();
		if (ServletFileUpload.isMultipartContent(request)) {
			FileItemFactory factory = new DiskFileItemFactory();
			ServletFileUpload upload = new ServletFileUpload(factory);
			List<FileItem> items = null;
			try {

				items = upload.parseRequest(request);
			} catch (Exception e) {
			}
			Iterator<FileItem> itr = items.iterator();
			while (itr.hasNext()) {
				item = itr.next();
				if (item.isFormField()) {
					String name = item.getFieldName();

					if (name.equals("gallery_name"))
						dto.setGallery_name(item.getString() == null ? ""
								: item.getString().trim());

					if (name.equals("gallery_id_pk"))
						dto.setGallery_id_pk(Integer
								.parseInt(item.getString() == null ? "0" : item
										.getString().trim()));

					if (name.equals("gallery_status"))
						dto.setGallery_status(item.getString() == null ? ""
								: item.getString().trim());
				} else {
					if (item.getSize() != 0) {
						if (item.getSize() < 1000000) {
							if (dto.getGallery_id_pk() == 0) {

								savedFile = new File(path + "GalleryImages/"
										+ service.maxEvent(request) + ".jpg");
								try {
									item.write(savedFile);
									System.out.println(savedFile);
								} catch (Exception e2) {
									e2.printStackTrace();
								}
							} else {
								try {
									savedFile = new File(path
											+ "GalleryImages/"
											+ dto.getGallery_id_pk() + ".jpg");

									item.write(savedFile);
								} catch (Exception e2) {
									e2.printStackTrace();
								}
							}
						}
					}

				}
			}
		}
		boolean b = false;
		if (dto.getGallery_id_pk() == 0) {
			System.out.println(dto.toString());
			b = service.addGallery(dto);
			if (b) {
				response.sendRedirect("add_gallery.jsp?msg=Data Insert Sucessfully");
			} else {
				response.sendRedirect("add_gallery.jsp?msg=Plz try again");
			}
		} else {
			b = service.updateGallery(dto);

			if (b) {
				response.sendRedirect("manage_gallery.jsp?msg=Data Update Sucessfully");
			} else {
				msg = "Plz try again";
				response.sendRedirect("edit_gallery.jsp?slider_id_pk="
						+ dto.getGallery_id_pk() + " " + msg);
			}
		}
	}

}
